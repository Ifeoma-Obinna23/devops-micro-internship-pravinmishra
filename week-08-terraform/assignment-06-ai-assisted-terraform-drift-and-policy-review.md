# Assignment 6 — AI-Assisted Terraform Drift and Policy Review

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Student Details

**Full Name:** Ifeoma Akabueze  
**GitHub Repository/Folder URL:** Add your GitHub URL here

---

## Purpose

Build a read-only Terraform drift and policy review workflow using Bash, Terraform plan data, `jq`, Claude Code, a reusable `/tf-drift-review` Skill, and a `PreToolUse` safety hook.

The workflow must follow this pattern:

```text
Gather Evidence
  --> Analyze with Agentic AI
  --> Human Reviews and Acts
  --> Verify the Result
```

The `/tf-drift-review` Skill and `tf-drift-check.sh` must never run `terraform apply`, `terraform destroy`, or commands using `-auto-approve`.

---

# Task 1 — Confirm the Clean Baseline and Create the Workspace

## Goal

Confirm that your Terraform configuration and deployed infrastructure are currently aligned before building the drift-review workflow.

## Evidence

### Screenshot 1 — Clean Terraform Plan

Add a screenshot of `terraform plan` showing no pending changes.

![alt text](screenshots/Clean-Terraform-Plan1-ass6.png)

![alt text](screenshots/Clean-Terraform-Plan2-ass6.png)

![alt text](screenshots/Clean-Terraform-Plan3-ass6.png)

---

### Screenshot 2 — Assignment Workspace

Add a screenshot of the folder structure showing `AI Assignment/`, `reports/`, and the Terraform project.

![alt text](screenshots/Assignment-Workspace-showing-AI-Assignment-reports-and-the-Terraform-project-ass6.png)

## Questions

### 1. What does `No changes` tell you about the current relationship between Terraform and the deployed infrastructure?

No changes” means three things are currently in sync: what I’ve written in my .tf files, what the state file last recorded, and what’s actually sitting in my AWS account right now. Terraform checks the third one live, during the “Refreshing state...” step, by calling the AWS API directly. it doesn’t just trust the state file blindly. When all three line up, there’s nothing for Terraform to reconcile.

I actually ran into the opposite of this earlier while recovering from a deleted state file — my .tf files described a full three-tier app, but the state file had zero resources in it, even though I’d need to check whether anything was still running in AWS. That mismatch is exactly what “No changes” is the absence of, and it’s why confirming this clean baseline before starting the drift-review script matters, I need to know any difference the script detects later is a real, deliberate change, not leftover confusion from before.

### 2. Why is a clean baseline important before introducing a test change?

A clean baseline matters because it isolates cause and effect. If terraform plan is clean before I make a test change, then anything it reports afterward can only be the result of that one change, not something else already in motion. I actually ran into the opposite of this during my own state-file recovery: I ran terraform plan and it showed a long list of changes, and I genuinely couldn’t tell whether those were real, intended changes or leftover drift from infrastructure that had already been destroyed. Without a clean starting point, I had no way to separate “what I just did” from “what was already wrong before I started.” That’s exactly the kind of ambiguity a clean baseline is meant to prevent.

---

# Task 2 — Create Project Context and Safety Rules in `CLAUDE.md`

## Goal

Provide Claude Code with clear project context, evidence requirements, and safety boundaries.

## Evidence

### Screenshot 3 — Project Context and Safety Rules

Add a screenshot of `CLAUDE.md` open in VS Code showing the Project Overview, Review Workflow, Safety Rules, and Output Rules.

![alt text](screenshots/CLAUDEmd-open-in-VSCode-showing-Project-Overview1-ass6.png)

![alt text](screenshots/CLAUDEmd-open-in-VSCode-showing-Project-Overview2-ass6.png)

![alt text](screenshots/CLAUDEmd-open-in-VSCode-showing-Project-Overview3-ass6.png)

![alt text](screenshots/CLAUDEmd-open-in-VSCode-showing-Project-Overview4-ass6.png)

![alt text](screenshots/CLAUDEmd-open-in-VSCode-showing-Project-Overview5-ass6.png)

![alt text](screenshots/CLAUDEmd-open-in-VSCode-showing-Project-Overview6-ass6.png)

![alt text](screenshots/CLAUDEmd-open-in-VSCode-showing-Project-Overview7-ass6.png)

## Questions

### 1. Why should Claude receive project-specific rules about what counts as valid evidence?

Without a project-specific evidence rule, Claude could sound confident without being right. An AI model is good at producing fluent, well-structured answers, but fluent isn’t the same as verified. A project-specific evidence rule closes that gap by defining exactly what counts as proof (the drift report, not Claude’s general training knowledge, not its assumptions). Again, different projects need different definitions of evidence. In this assignment, evidence means the Bash script’s output. In a different context, it might mean something else entirely. Generic AI safety instructions can’t know that except me, writing project-specific rules, can tell Claude exactly where the ground truth lives for this workflow.

### 2. Why must the human remain responsible for running `terraform apply`?

A human must remain responsible for running terraform apply because it makes real changes to real infrastructure unlike plan, which only predicts. The Agentic Loop has four phases, and I believe this is exactly why the human stays in control of that one step: Gather, where Bash scripts run read-only commands only; Analyze, where Claude interprets the evidence, identifies a likely cause, and proposes a fix; Human Act, where the human reviews that proposal and decides whether to actually run it; and Verify, where the result is checked afterward rather than just trusted.

I saw why this matters directly today. When terraform plan showed 67 resources to add, I had to recognize that as something to investigate rather than approve automatically. if I’d just typed yes, I would have risked standing up a second, duplicate copy of my entire infrastructure alongside resources that might still have existed, instead of discovering the real cause (an empty state file). And when a Secrets Manager command reported success, I still had to check the actual region in its output myself. The deletion had silently gone to eu-west-2 instead of us-east-1, meaning an automated process that only trusted the “success” message would have left the real blocking secret untouched and kept failing for a reason no one would have understood.

### 3. Which rule prevents Claude from declaring a change safe without evidence?

The rule 4 that says - "Claude must not claim a change is safe to apply unless
  `reports/tf-drift-report.txt` actually contains evidence supporting that conclusion"

---

# Task 3 — Build the Terraform Drift and Policy Check Script

## Goal

Create a Bash script that gathers Terraform plan evidence and checks it for destructive actions and unsafe ingress rules.

## Evidence

### Screenshot 4 — Script Variables and Checks Array

Add a screenshot of the top section of `tf-drift-check.sh` showing the variables and `checks` array.

![alt text](screenshots/top-section-of-tf-drift-checksh-showing-the-variables-and-checks-array-ass6.png)

---

### Screenshot 5 — Destructive-Action and Open-Ingress Checks

Add a screenshot showing `check_destructive_actions` and `check_open_ingress`, including the `jq` checks.

![alt text](screenshots/Destructive-Action-and-Open-Ingress-Checks-ass6.png)

![alt text](screenshots/Open-Ingress-Checks-ass6.png)

---

### Screenshot 6 — Script Validation and Permissions

Add a screenshot showing successful `bash -n` and `ls -l` output.

![alt text](screenshots/Script-Validation-and-Permissions-ass6.png)

## Questions

### 1. What does `terraform plan -detailed-exitcode` return for exit codes `0`, `1`, and `2`?

check_plan_exit_code() {
  case "$plan_exit_code" in

    0)
      mark_pass "terraform plan exit code 0 — no pending changes"
      ;;

    1)
      mark_failure "terraform plan exit code 1 — Terraform plan failed"
      ;;

    2)
      mark_warning "terraform plan exit code 2 — changes are pending and require review"
      ;;

      Exit code 0 means Terraform ran the plan successfully and found no differences between my .tf files and the real infrastructure. My check_plan_exit_code function marks this as a PASS. Exit code 1 means the plan itself failed to complete, due to something like a syntax error or an authentication problem, before Terraform could even determine whether changes exist. My script marks this as a FAIL. Exit code 2 means the plan ran successfully and found pending changes that haven’t been applied yet, this isn’t an error, just a signal that something differs and needs review, so my script marks it WARN rather than FAIL.


### 2. Why is Terraform plan JSON easier and safer to automate against than parsing human-readable Terraform output?

Human-readable Terraform output is formatted for a person to read — symbols, indentation, colored text — and if I tried to script against it, I’d have to use fragile text-matching to guess what each line means, which can break silently if Terraform’s wording changes between versions. The JSON output from terraform show -json is structured, predictable data with a stable schema, so I can ask precise questions against it with jq, like “give me every resource where the actions array contains delete,” instead of guessing from prose. This matters especially here because my script is a security tool — if it silently misread human-readable text and missed a destructive change, that failure would be dangerous. Every check in my script queries the JSON directly; I never had to write a regex against Terraform’s text output.

### 3. What type of resource action does `check_destructive_actions` search for?

It searches the plan JSON for any resource whose change.actions array contains "delete", using jq. That one field catches both outright deletions and replacements, since Terraform represents a replacement as a resource being deleted and a new one created in its place.

### 4. Why does finding a `delete` action also help detect replacements?

In Terraform’s plan JSON, a resource replacement isn’t a separate action type — it’s represented as the same resource showing "delete" alongside "create" in its actions array, because the old resource is destroyed and a new one is created to take its place. So by checking only for "delete", my script automatically catches replacements too, without needing a second, separate check for them.

### 5. Why must this script never run `terraform apply`?

This script’s entire purpose is to gather and report evidence for a human to review before any infrastructure-changing decision is made — the moment it ran apply itself, it would collapse the Gather and Act phases into one automated step, removing the human review the whole workflow depends on. It’s also built specifically as a read-only tool: every command it runs (terraform plan, terraform show -json, jq queries) only reads information, and adding apply would turn a safe inspection tool into something that could make real, costly, or destructive changes to AWS without anyone having actually looked at the plan first.

---

# Task 4 — Run the Script Against the Clean Baseline

## Goal

Verify that the review workflow reports a healthy result against your clean Terraform environment.

## Evidence

### Screenshot 7 — Healthy Baseline Report

Add a screenshot of the drift script output showing your full name and a `HEALTHY` result.

![alt text](screenshots/Healthy-Baseline-Report-ass6.png)

---

### Screenshot 8 — Baseline Script Exit Code

Add a screenshot showing the captured script exit code `0`.

![alt text](screenshots/Baseline-Script-Exit-Code-ass6.png)

## Questions

### 1. What is the Overall Status of your baseline?

HEALTHY

### 2. Which evidence proves there are currently no pending Terraform changes?

"No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no
changes are needed."

### 3. Was `reports/tfplan.json` created? Explain why or why not.

No, reports/tfplan.json was not created. My script only converts the plan to JSON when plan_exit_code equals 2 — meaning Terraform found pending changes to inspect. Since my baseline returned exit code 0 (no changes), there’s nothing for the destructive-actions or open-ingress checks to analyze, so the script correctly skips generating that file rather than wastefully converting an empty plan.

---

# Task 5 — Create and Run the `/tf-drift-review` Claude Code Skill

## Goal

Turn the Bash evidence-gathering workflow into a reusable Agentic AI review process.

## Evidence

### Screenshot 9 — `/tf-drift-review` Skill Configuration

Add a screenshot of `SKILL.md` showing the frontmatter, allowed tools, and safety rules.

![alt text](screenshots/SKILLmd-showing-the-frontmatter-allowed-tools-and-safety-rules-ass6.png)


---

### Screenshot 10 — Clean Agentic AI Review

Add a screenshot of `/tf-drift-review` showing the clean `HEALTHY` result.

![alt text](screenshots/tf-drift-review-showing-the-clean-HEALTHY-result-ass6.png)

![alt text](screenshots/tf-drift-review-showing-the-clean-HEALTHY-result2-ass6.png)



## Questions

### 1. Why does this Skill have `Bash`, `Read`, and `Grep`, but not `Write`?

This Skill only needs to run the drift-check script and read its output — it never needs to create or change a file, so Write isn’t included. This isn’t just a stated rule, it’s an actual enforced limit: even if Claude reasoned its way toward wanting to edit something, it couldn’t, because the tool to do so isn’t available in this Skill’s permissions. That backs up the Safety Rule I wrote in CLAUDE.md saying Claude must not edit Terraform files during a drift review — the missing Write tool is what makes that rule real instead of just a request.

### 2. Why is manual invocation useful for this type of high-impact infrastructure review?

Manual invocation matters because disable-model-invocation: true means Claude can’t decide on its own, mid-conversation, to run this Skill — a human has to deliberately type /tf-drift-review for it to run at all. For a high-impact review like this one, that matters because it keeps a person in control of when infrastructure gets checked, not just what happens during the check itself. If Claude could trigger this automatically whenever it judged a review “seemed useful,” that would be a small step away from the human-in-the-loop principle the whole assignment is built on — the review is supposed to happen because an engineer chose to ask for it at a specific moment, not because the AI decided on its own that now was a good time to look.

### 3. Which part of the workflow is deterministic Bash automation?

The deterministic part is everything inside tf-drift-check.sh itself — running terraform plan -detailed-exitcode, checking the exit code with a case statement, and querying the resulting JSON with jq for delete actions and open ingress rules. Given the same input (the same Terraform configuration and the same real infrastructure), this script will produce the exact same PASS/WARN/FAIL result every single time — there’s no interpretation or judgment involved, just fixed logic checking fixed conditions.

### 4. Which part requires Claude's reasoning?

The part that requires Claude’s reasoning is turning the raw PASS/WARN/FAIL lines and exit code into something a human can actually act on — the plain-language risk assessment, explaining why a finding matters, and recommending a next step. When I ran /tf-drift-review, the script gave fixed output like [PASS] terraform plan exit code 0, but Claude is the one that explained what that meant in context — that the live infrastructure matches the configuration, that nothing needs to be applied, and that it’s safe to move on. That interpretation isn’t scripted anywhere; it has to be reasoned out each time based on what the evidence actually shows.

### 5. Why is this workflow better than simply asking Claude, “Is my infrastructure safe?”

This workflow is better because it forces Claude to ground its answer in specific, verifiable evidence instead of just reasoning generally from the question. If I’d simply asked “is my infrastructure safe?” with no script behind it, Claude could answer based on plausible-sounding general knowledge about what usually looks safe — without ever actually checking my real Terraform plan or my real AWS account. With this workflow, Claude has no choice but to run tf-drift-check.sh first, read the actual reports/tf-drift-report.txt it produces, and point to specific evidence like the exit code or a PASS/FAIL count before saying anything. The answer is tied to something I can go verify myself line by line, not just to Claude sounding confident.

---

# Task 6 — Introduce a Controlled Difference and Detect It

## Goal

Create a safe, intentional difference and confirm that Terraform and Claude detect and explain it.

## Evidence

### Screenshot 11 — Controlled Difference

Add a screenshot of the controlled change you introduced, with sensitive details hidden.

![alt text](screenshots/The-controlled-change-introduced-ass6.png)

![alt text](screenshots/The-controlled-change-introduced2-ass6.png)

---

### Screenshot 12 — Detected Difference and Risk Assessment

Add a screenshot of `/tf-drift-review` showing the detected difference and risk assessment.

![alt text](screenshots/tf-drift-review-showing-the-detected-difference-and-risk-assessment-ass6.png)

![alt text](screenshots/tf-drift-review-showing-the-detected-difference-and-risk-assessment2-ass6.png)

---

### Screenshot 13 — Detected Drift Report

Add a screenshot of `drift-detected-report.txt` showing your full name and the `WARN` or `FAIL` result.

![alt text](screenshots/Drift-detected-report-txt-showing-your-full-name-and-the-FAIL-result-ass6.png)

## Questions

### 1. What change did you introduce?

I added a non-destructive configuration change to terraform/modules/network/main.tf: a new Owner = "Ifeoma Jane Akabueze" tag on the aws_vpc.main resource, alongside the existing Name tag.

### 2. Was it true infrastructure drift or a Terraform configuration change?

This is a configuration change, not drift. True drift happens when the real AWS infrastructure changes outside of Terraform (e.g., someone edits a resource manually in the console) so that the live state no longer matches the .tf files. Here, the opposite happened: I edited the .tf file itself, so Terraform’s desired state now differs from the deployed state. Running terraform plan surfaces this the same way it would surface real drift — as a difference between desired and actual state — which is why it’s a useful stand-in for testing detection, even though the direction of the mismatch is different.

### 3. What Terraform plan evidence proves that a change is pending?

The plan output shows module.network.aws_vpc.main will be updated in-place, with ~ tags showing + "Owner" = "Ifeoma Jane Akabueze" being added, and a summary line of Plan: 0 to add, 1 to change, 0 to destroy. The detailed exit code returned 2, confirming pending changes exist (as opposed to 0 for no changes or 1 for an error).

### 4. Was the action an update, deletion, replacement, or security-rule change?

It’s a change (in-place update) — not an add or a destroy. Risk level is low: it only affects a resource tag, which is metadata used for cost allocation/ownership tracking and has no effect on networking, security, or availability

### 5. What did Claude recommend?

The skill reported an overall status of FAIL — driven by its open-ingress check flagging the public ALB’s security group (ports 80/443 from 0.0.0.0/0). On inspection, this was a false positive: that rule is a no-op in the plan (not an actual change) and is the explicitly allowed exception in the project’s security rules for the public-facing load balancer. The only real pending change was the VPC tag, which the skill separately confirmed was non-destructive. The skill’s recommendation was that applying the tag change looks safe based on the evidence, but it deliberately did not run terraform apply itself — consistent with the project’s human-in-the-loop rule that only a human approves and executes changes — and left that decision and action to me.

### 6. Why should you review the recommendation before taking action?

Because even a “safe-looking” plan can hide details a script’s heuristics miss, and because applying infrastructure changes has real cost and availability consequences — an automated tool acting without review risks silently applying something destructive or insecure. Human-in-the-loop review ensures a person checks the actual plan output, understands why each flagged item is or isn’t a real risk (like the false-positive open-ingress here), and makes the final, accountable decision — which is also why this project’s rules explicitly forbid any AI assistant or script from auto-running terraform apply or destroy.

---

# Task 7 — Add a `PreToolUse` Hook to Block Unsafe Apply Attempts

## Goal

Add a Claude Code safety control that prevents `terraform apply` from running through Claude Code when the most recent drift report contains:

```text
Overall Status: FAIL
```

## Evidence

### Screenshot 14 — `PreToolUse` Safety Hook

Add a screenshot of `.claude/settings.json` showing the `PreToolUse` safety hook.

![alt text](screenshots/Claude-settingsjson-showing-the-PreToolUse-safety-hook-ass6.png)

---

### Screenshot 15 — Blocked Apply Attempt

Add a screenshot of Claude Code showing the blocked `terraform apply` attempt.

![alt text](screenshots/ClaudeCode-showing-the-blocked-terraform-apply-attempt-ass6.png)

![alt text](screenshots/ClaudeCode-showing-the-blocked-terraform-apply-attempt1-ass6.png)

![alt text](screenshots/ClaudeCode-showing-the-blocked-terraform-apply-attempt2-ass6.png)

## Questions

### 1. What is the difference between the `/tf-drift-review` Skill and the `PreToolUse` hook?

/tf-drift-review is advisory and on-demand. It’s a Skill I invoke manually, as a slash command, whenever I want an analysis. It gathers evidence (reads the drift report, runs terraform show -json, inspects the plan) and produces a human-readable risk assessment — summarizing what changed, flagging destructive actions or open ingress, and recommending whether apply looks safe. But it has no enforcement power: it only informs a decision. Nothing stops me (or anyone) from running terraform apply immediately afterward, ignoring its recommendation entirely.
The PreToolUse hook is enforcing and automatic. It’s not something I choose to run — it fires on every single Bash tool call I attempt, without exception, checking whether that command matches terraform apply/destroy. If it does, the hook reads the latest drift report itself and, if the status is FAIL, blocks the tool call outright (exit code 2) before it ever executes — regardless of what I’ve concluded, recommended, or been asked to do in the conversation.

### 2. Which component performs analysis?

The /tf-drift-review Skill performs the analysis. It’s the component that actually reasons about the situation: it reads the drift report and plan output, interprets what changed, distinguishes real risk from false positives, and produces a plain-language recommendation.

### 3. Which component enforces the safety gate?

The PreToolUse hook enforces the safety gate. It’s a simple, deterministic check (a shell script reading one line from a text file: Overall Status: FAIL or not) that either allows or blocks a command. It doesn’t understand why something failed, weigh severity, or reason about context; it just enforces a binary gate based on the last report’s status. That’s intentional: enforcement should be simple and predictable enough that it can’t be reasoned around, while analysis is where the nuanced judgment belongs.

### 4. Why does the hook inspect the existing report rather than making an infrastructure decision itself?

Because those are two fundamentally different kinds of work, and mixing them into one component would weaken both.

Deciding whether a piece of infrastructure drift is actually risky requires judgment — understanding Terraform plan output, recognizing which resources matter, knowing the project’s security conventions (like the public ALB’s intentional 0.0.0.0/0 exception), and telling a genuine risk apart from a false positive. That’s analysis, and it belongs in the /tf-drift-review Skill, where it can be as thorough and context-aware as needed, and can be re-run, refined, or second-guessed without touching the safety mechanism itself.

A PreToolUse hook, by contrast, needs to be simple, fast, and deterministic enough to trust blindly, every single time, with no exceptions. If the hook tried to re-analyze the Terraform plan itself — parsing JSON, applying heuristics, making judgment calls — it would inherit all the same failure modes as any analysis step (false positives, missed edge cases, or being “argued around” the way the model was pushed to reconsider earlier). A gate that can be reasoned with isn’t really a gate.

By having the hook simply check the already-computed status from the most recent report, it stays dumb and reliable — a single line of text, matched exactly — while letting the smarter work happen upstream, under separate review, on a timeline the human controls (you decide when to re-run the analysis and resolve findings; the hook just enforces whatever that analysis last concluded). This separation is also what makes the hook resistant to prompt-based persuasion: there’s no reasoning inside it to persuade.

### 5. Why is a deterministic guard useful for high-impact commands?

Because high-impact commands like terraform apply and destroy especially, have consequences that are expensive or impossible to undo (dropped databases, deleted networking, real AWS billing), so the control guarding them needs to be as trustworthy as the risk is large. A deterministic guard gives you three things a judgment-based one can’t:

Predictability. It behaves the exact same way every single time, given the same input. There’s no variance from phrasing, mood, context window, or how the request is framed — which matters because an inconsistent guard is really just a guard that sometimes fails.
Resistance to persuasion. A rule enforced through reasoning (even a model’s own careful judgment, or a person’s own good intentions under pressure) can be talked around — reframed as “just testing,” “a special case,” or argued down over the course of a conversation, exactly as nearly happened a few minutes ago. A deterministic check has nothing to persuade; it either reads FAIL and blocks, or it doesn’t. That makes it resistant to both social engineering and well-meaning but mistaken exceptions.
Verifiability. Because the logic is simple and fixed, you can test it directly and know exactly what it will do — which is literally what we did by piping a fake payload straight into the script. A human-judgment-based control can’t be tested that cleanly, because its behavior isn’t fixed.

In short: for low-stakes decisions, flexible judgment is an asset. For high-stakes, hard-to-reverse ones, the same flexibility becomes the vulnerability — so the safest design pushes judgment upstream (into analysis, where it can be nuanced) and keeps the actual gate simple, rigid, and incorruptible.

---

# Task 8 — Resolve the Difference and Verify the Final State

## Goal

Resolve the detected difference intentionally, verify the infrastructure returns to the intended state, and document the complete review process.

## Evidence

### Screenshot 16 — Human-Reviewed Resolution

Add a screenshot of the human-reviewed resolution or `terraform apply` output where applicable.

![alt text](screenshots/Human-reviewed-terraform-apply-output-ass6.png)

![alt text](screenshots/Human-reviewed-terraform-apply-output1-ass6.png)

---

### Screenshot 17 — Final Healthy Review

Add a screenshot of the final `/tf-drift-review` showing `HEALTHY`.

![alt text](screenshots/final-tf-drift-review-showing-HEALTHY-ass6.png)

---

### Screenshot 18 — Saved Reports

Add a screenshot of `ls -lah reports` showing both:

- `drift-detected-report.txt`
- `resolved-report.txt`

![alt text](screenshots/Screenshot-of-ls-lah-reports-showing-ass6.png)

---

### Screenshot 19 — Drift Review Summary

Add a screenshot of `drift-review-summary.md` showing all required sections and your full name.

![alt text](screenshots/Drift-review-summarymd-showing-all-required-sections-and-full-name-ass6.png)

![alt text](screenshots/Drift-review-summarymd-showing-all-required-sections-and-full-name1-ass6.png)

![alt text](screenshots/Drift-review-summarymd-showing-all-required-sections-and-full-name2-ass6.png)

## Terraform Drift Review Summary

### 1. Change Introduced

Explain the controlled change you introduced.

State whether it was:

- True infrastructure drift, or
- A Terraform configuration change

I introduced a controlled, intentional configuration change: adding a new `Owner = "Ifeoma Jane Akabueze"` tag to the `aws_vpc.main` resource in `terraform/modules/network/main.tf`, alongside the existing `Name` tag.

This was:
- A Terraform configuration change, not true infrastructure drift.

True drift happens when the real AWS resources change outside of Terraform (e.g. someone edits something manually in the console), so the live state no longer matches the `.tf` files. Here it was the reverse: I edited the `.tf` file itself, so Terraform's desired state temporarily differed from the deployed state. This still produces the same kind of detectable mismatch when `terraform plan` runs, which is why it made a safe, controlled stand-in for testing the detection and review workflow.


### 2. Evidence Collected

Describe the Terraform plan evidence and affected resource.

Running `terraform plan -detailed-exitcode` showed `module.network.aws_vpc.main will be updated in-place`, with the `tags` map gaining `+ "Owner" = "Ifeoma Jane Akabueze"`, and a summary line of `Plan: 0 to add, 1 to change, 0 to destroy`. The detailed exit code returned `2`, confirming a pending, non-destructive change to exactly one resource — the VPC.

### 3. Risk Assessment

Explain the risk identified by the Bash check and Claude Code.

The Bash check (`tf-drift-check.sh`) and `/tf-drift-review` agreed the actual pending change — the tag — was low-risk: a metadata-only update with no effect on networking, security, or availability. Separately, the first run of the check returned an overall `FAIL`, but this was a false positive in its `check_open_ingress` function: it scanned every resource in the plan's JSON regardless of whether it was actually changing, so it flagged the public ALB's pre-existing, intentionally public listener ports (HTTP/HTTPS from `0.0.0.0/0`) even though nothing about that resource was part of this plan's diff. I fixed the script to skip resources whose `change.actions` is `["no-op"]` before checking for open ingress, so it now only flags genuinely new or changed exposure rather than long-standing allowed rules.

### 4. Human-Approved Action

Explain the action you reviewed and executed manually.

After reviewing the Terraform plan and Claude's analysis, I ran `terraform apply "tfplan.out"` manually in my own terminal — applying the exact plan that had already been reviewed and saved, rather than re-planning at apply time. This was a deliberate human decision and action; at no point did Claude or any script run `apply` or `destroy` automatically. The output confirmed `Apply complete! Resources: 0 added, 1 changed, 0 destroyed`.

### 5. Verification

Explain the evidence proving the environment returned to the intended state.

After applying, I ran `/tf-drift-review` again. It reported `Overall status: HEALTHY` with 3 PASS, 0 WARN, 0 FAIL, and confirmed `terraform plan`'s detailed exit code was `0` — meaning Terraform refreshed the real AWS state and found no differences from the configuration. I saved this as `reports/resolved-report.txt` and confirmed both it and the earlier `drift-detected-report.txt` exist side by side in `reports/`.

### 6. Safety Decision

Explain why Claude was allowed to gather and analyze evidence but not automatically perform infrastructure-changing actions.

Claude was allowed to gather evidence (reading plan output, running `tf-drift-check.sh`, inspecting the plan JSON) and analyze it (classifying risk, explaining findings) because that work benefits from speed and thoroughness, and a mistake there only costs a re-run. Claude was never allowed to execute `terraform apply` or `destroy`, because those actions are difficult or impossible to reverse and carry real cost and availability consequences — a mistaken or persuaded automated action there is far more expensive than a mistaken analysis. This is enforced two ways: as an explicit rule in `CLAUDE.md` that Claude follows through judgment, and as a structural, deterministic `PreToolUse` hook in `.claude/settings.json` that blocks any `apply`/`destroy` attempt whenever the latest drift report shows `FAIL` — regardless of how the request is phrased or justified in conversation. I confirmed this works both by testing the hook directly and by watching Claude Code itself decline to run `apply` even when asked to "just demonstrate" the block

### 7. Agentic Loop Mapping

Explain how your workflow followed:

```text
Gather --> Analyze --> Human Act --> Verify
```
- Gather: `tf-drift-check.sh` ran `terraform plan -detailed-exitcode` and `terraform show -json` to collect raw evidence about the pending change.
- Analyze: The script classified that evidence into PASS/WARN/FAIL checks, and `/tf-drift-review` added a plain-language risk assessment, explaining what was safe and identifying the open-ingress check's false positive.
- Human Act: I reviewed the plan and the analysis myself, decided the tag change was safe and intentional, and ran `terraform apply "tfplan.out"` manually — the one step no AI component was permitted to perform.
- Verify: I re-ran `/tf-drift-review` after applying to confirm the environment returned to `HEALTHY`, with Terraform's own plan output as independent proof that reality now matches configuration.

## Questions

### 1. What action did you execute to resolve the difference?

I ran `terraform apply "tfplan.out"` manually in my own terminal, applying the exact, already-reviewed plan that added the `Owner` tag to `aws_vpc.main`.

### 2. Did you review `terraform plan` before taking action?

Yes. I reviewed the full plan output beforehand, confirming it showed only an in-place tag update (`~ update in-place`, `Plan: 0 to add, 1 to change, 0 to destroy`) with no additions, deletions, or replacements, before running `apply`.

### 3. What evidence proves the environment is now aligned?

Re-running `/tf-drift-review` after the apply showed `Overall status: HEALTHY` (3 PASS, 0 WARN, 0 FAIL), with Terraform's own `plan -detailed-exitcode` returning `0` — meaning it refreshed the real AWS state and found it matches the configuration exactly, with nothing pending.

### 4. Why is a second drift review required after the fix?

Because applying a change is not, by itself, proof that it succeeded correctly or that nothing else was affected. A second review independently re-checks the real infrastructure against the configuration using the same evidence-based process as the first — closing the loop and confirming the actual outcome, rather than just trusting that the apply command "should have" worked.

### 5. What could go wrong if an AI agent automatically applied every detected Terraform change?

It could apply changes that look routine but are actually destructive (deleting or replacing a resource, like a database, with real data loss), silently widen security exposure (like opening an ingress rule), or compound a mistake across many resources before anyone notices — all without a human check catching the error beforehand. It would also remove the chance to catch tool-level false positives/negatives (like the open-ingress bug found in this exercise) before they cause real consequences.

### 6. In one sentence, explain the difference between asking an AI chatbot “Is my infrastructure okay?” and using this evidence-based Agentic AI workflow.

Asking a chatbot "is my infrastructure okay?" gets an answer based on whatever it assumes or recalls, with no way to verify it reflects your actual current state, while this workflow produces its answer only after running real commands against your real infrastructure, capturing that evidence in a report, and requiring a human to review and act on the risky step — making the conclusion verifiable and the execution accountable.

---

# LinkedIn Post — Mandatory

## Goal

Publish a LinkedIn post in your own words describing:

- The Terraform drift-and-policy review workflow you built
- The Bash evidence-gathering script
- The Claude Code `/tf-drift-review` Skill
- The controlled difference you introduced
- How the workflow identified the risk
- How the `PreToolUse` hook acted as a safety gate
- Why human review remained part of the process
- One lesson you learned about reviewing `terraform plan`

Include a screenshot of the detected change and a screenshot of the final `HEALTHY` review in your post.

Suggested tags:

```text
#DMIByPravinMishra #Terraform #AgenticAI #ClaudeCode #DevOps
```

## LinkedIn Evidence

### LinkedIn Post URL

https://lnkd.in/p/eR44b7MQ

### Published LinkedIn Post Screenshot — Mandatory

![alt text](screenshots/The-published-LinkedIn-post-ass6.png)

---

# Required Assignment Files

Confirm that the following files are included in your GitHub repository:

- `CLAUDE.md`
- `AI Assignment/tf-drift-check.sh`
- `.claude/skills/tf-drift-review/SKILL.md`
- `.claude/settings.json` containing the safety hook
- `reports/drift-detected-report.txt`
- `reports/resolved-report.txt`
- `drift-review-summary.md`

---

# Submission Instructions

- Complete Tasks 1–8 in sequence.
- Include Screenshots 1–19 exactly as specified.
- Answer every question under Tasks 1–8 in your own words.
- Complete all seven sections of the Terraform Drift Review Summary.
- Include the GitHub repository/folder URL containing the assignment files.
- Include your full name in the required reports and screenshots.
- Include the LinkedIn post URL and a screenshot of the published LinkedIn post.
- Do not expose access keys, passwords, tokens, account IDs, private keys, Terraform secrets, or other sensitive information.
- Review all screenshots carefully and hide or redact sensitive details where necessary.

---

# Completion Checklist

- [ ] Confirmed a clean Terraform baseline
- [ ] Created the required assignment workspace
- [ ] Created or updated `CLAUDE.md`
- [ ] Added project context and safety rules
- [ ] Created `tf-drift-check.sh`
- [ ] Added my full name to the report
- [ ] Validated the Bash script
- [ ] Made the script executable
- [ ] Used `terraform plan -detailed-exitcode`
- [ ] Used Terraform plan JSON
- [ ] Used `jq` to inspect destructive actions
- [ ] Used `jq` to inspect unsafe ingress
- [ ] Confirmed the baseline returns `HEALTHY`
- [ ] Created `/tf-drift-review`
- [ ] Restricted the Skill to appropriate tools
- [ ] Confirmed the Skill remains read-only
- [ ] Confirmed the Skill never runs `terraform apply`
- [ ] Confirmed the Skill never runs `terraform destroy`
- [ ] Introduced a controlled detectable difference
- [ ] Correctly identified whether it was true drift or a configuration change
- [ ] Saved `drift-detected-report.txt`
- [ ] Added the `PreToolUse` safety hook
- [ ] Verified the hook blocks `terraform apply` when the report is `FAIL`
- [ ] Reviewed the Terraform evidence before resolving the change
- [ ] Performed any infrastructure-changing action manually
- [ ] Ran the drift review again after resolution
- [ ] Confirmed the final status is `HEALTHY`
- [ ] Saved `resolved-report.txt`
- [ ] Completed `drift-review-summary.md`
- [ ] Mapped the workflow to `Gather --> Analyze --> Human Act --> Verify`
- [ ] Included all 19 numbered screenshots
- [ ] Answered all required questions
- [ ] Published the required LinkedIn post
- [ ] Added the LinkedIn post URL and screenshot
- [ ] Included the GitHub repository/folder URL
- [ ] Confirmed that no sensitive information is exposed

---

*This submission is part of the DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
