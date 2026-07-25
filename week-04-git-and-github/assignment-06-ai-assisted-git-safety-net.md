# Assignment 6 — Building an AI-Assisted Git Safety Net (PR Ready Check)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In Week 2 you built Claude Code hooks that block a dangerous action *before* it happens (`PreToolUse`), and a restricted skill that could look but not touch (`allowed-tools` without `Write`). In this assignment you will discover that Git has the exact same idea, decades older: a **pre-commit hook** that blocks a commit before it's created.

You will build both halves of a real "PR Ready" workflow:

1. A **Git hook that follows fixed rules** — scans staged changes for hardcoded secrets and oversized files and refuses the commit. No AI involved, no guessing, just a rule that gives the same answer every time.
2. A **restricted Claude Code skill** (`/pr-ready`) that reads your staged diff and drafts a Pull Request title, description, and a short list of things worth a second look — the kind of judgment a fixed rule can't make (mixed changes, missing context, unclear intent). The skill never commits, pushes, or opens the PR. You do that yourself, using its draft as a starting point.

This mirrors the Agentic Loop from Week 3's Linux triage assignment: **Gather → Analyze → Human Act → Verify**. The hook and the skill both gather and analyze; only you act.

---

# Task 0 — Confirm Your Fork and Create a Feature Branch

## Goal

Confirm you are working in your own fork, then create a dedicated branch for this assignment.

### Evidence

#### Screenshot 1 — Output of git remote -v and git branch showing the new branch

![alt text](screenshots/Output-git-branch-remote-ass6.png)

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

Working directly on main risks introducing incomplete or risky code into the branch everyone else builds from. A dedicated feature branch isolates this work so it can be reviewed and merged deliberately, and if something goes wrong, main is never at risk. It also lets you have multiple pieces of work in progress without them interfering with each other.

---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 1 — Output of  `git status` showing the staged file on feature/ai-pr-ready

![alt text](screenshots/git-status-showing-stagedfile-ass6.png)

---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

This assignment uses a clearly fake AWS key (AKI-AABCDEFGHIJKLMNOP) instead of a real one because the goal is to test whether the safety checks correctly detect and block secret-like patterns — not to actually handle sensitive credentials. Using a real key, even briefly, would create genuine risk: it could be accidentally pushed to a remote fork, exposed in a screenshot, cached in shell history, or picked up by GitHub's own secret-scanning bots before the local hook ever runs. A fake key that looks like a real credential (matching AWS's key-ID pattern) is enough to validate that the pre-commit hook's detection logic works, while keeping the entire exercise completely safe

---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 2 — `hooks/pre-commit` open in VS Code showing the full script

![alt text](screenshots/hooks-pre-commit-ass6.png)
---

#### Screenshot 3 — Output of `git config core.hooksPath` confirming it points to `hooks`

![alt text](screenshots/Output-git-config.core-ass6.png)

---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

.git/hooks/ is local to each person's machine. It's never committed, never pushed, and doesn't exist in the repository's history at all. If the hook only lived there, every teammate would have to know it exists, manually recreate it, and remember to keep it updated. Nothing would enforce that anyone actually has it enabled, a new clone starts with zero hooks.
By tracking the script in hooks/ and pointing core.hooksPath at it, the hook travels with the repository itself. Anyone who clones the repo and runs one git config core.hooksPath hooks command gets the exact same protection the rest of the team has — same rules, same detection patterns, same behavior. It turns a personal habit into a shared, version-controlled safety net that's visible, auditable, and consistent across the whole team.

---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**

PreToolUse intercepts an AI agent's tool call. It steps in right before the agent executes an action (like running a shell command or editing a file) and can block it based on fixed rules. hooks/pre-commit intercepts a Git commit — it steps in right before the commit is finalized and can block it based on fixed rules (secret patterns, file size).
What they have in common:

Both run automatically, without needing a human to remember to trigger them
Both sit at a "before it's permanent" checkpoint — after work has been done, but before it's committed to history or executed for real
Both enforce fixed, deterministic rules — no judgment, no interpretation, just pattern matching against a known risk
Both exist because relying on a human to catch every mistake manually doesn't scale — the check has to be baked into the workflow itself, not left as an optional step someone might skip

---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 4 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file

![alt text](screenshots/git-commit-attempt-blocked-ass6.png)
---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**

if git diff --cached -- "$file" | grep -qE 'AKIA[0-9A-Z]{16}|-----BEGIN (RSA|OPENSSH|PRIVATE) KEY-----'; then

Specifically, the AKIA[0-9A-Z]{16} portion of the regex matched the fake key AKI-AABCDEFGHIJKLMNOP. AWS access key IDs always start with the literal prefix AKIA followed by exactly 16 uppercase letters or digits. The fake key follows that exact shape — AKIA + ABCDEFGHIJKLMNOP (16 uppercase characters) — so even though the key isn't real, it matches the pattern a real AWS key would have. The hook isn't checking whether the key is valid or active; it's just checking whether the staged diff contains something that looks like a credential, which is exactly why a fake-but-realistically-shaped key is enough to trigger it.

---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No.  This hook would completely miss it. The regex only matches two very specific, recognizable patterns: the AWS access key prefix (AKIA...) and PEM-style private key headers (-----BEGIN ... KEY-----). If a secret were stored in a variable like token = "sk_live_9f8a7b...", db_password = "hunter2", or even secret_key = "AKIA..." written in lowercase or with extra characters breaking the exact pattern, the hook would find nothing wrong and let the commit through without a single warning.

---

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 5 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

![alt text](screenshots/Skill.md-showingallowed-bash-read-grep-ass6.png)

---

#### Screenshot 6 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement

![alt text](screenshots/pr-ready-output-ass6.png)

---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

Why does /pr-ready have Bash and Read but not Write?

/pr-ready's entire job is to observe and report — it needs Bash to run commands like git diff --cached and git status, and Read/Grep to inspect file contents and search for risky patterns. None of that requires changing anything on disk. Leaving Write out of allowed-tools isn't an accident — it's a deliberate, structural boundary that makes it impossible for the skill to edit files, even if it wanted to, even if a prompt injection or a misinterpreted instruction tried to push it that way. The permission model enforces the design intent at the tool level, not just as a written instruction the AI is trusted to follow. This keeps the human as the only one who can actually act — commit, push, or open the PR — while the AI's role stays limited to advising. It's the same principle behind giving someone "read-only" access to a system: the restriction isn't about trust in behavior, it's about removing the capability to do harm in the first place.

---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

No. They didn't flag the same things. The pre-commit hook only caught the hardcoded-looking AWS key (AKIA...), because that's the one pattern its regex was explicitly written to match. It said nothing about the debug statement, since echo "DEBUG: token is $AWS_ACCESS_KEY_ID" doesn't match any secret-shaped pattern. The hook has no concept of "this line prints something sensitive," only "does this string match a known key format."

/pr-ready, on the other hand, caught both issues — the credential-shaped string and the leftover debug echo — and went a step further, noting that the change had no accompanying explanation of what notify.sh was for. That third observation is something a fixed-rule hook could never make, since it requires understanding the purpose and context of the change, not just scanning its content for a pattern
---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 7 — `git commit` succeeding after the fix (no BLOCKED message)

![alt text](screenshots/git-commit-after-fix-ass6.png)

---

#### Screenshot 8 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description

![alt text](screenshots/pr-ready-output-after-fix-ass6.png)

---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**

I removed the two lines in scripts/notify.sh that were triggering the hook's checks:

AWS_ACCESS_KEY_ID=AKIAABCDEFGHIJKLMNOP — this matched the hook's regex (AKIA[0-9A-Z]{16}), which is what caused the BLOCKED: possible secret message on the first attempt.
echo "DEBUG: token is $AWS_ACCESS_KEY_ID" — this printed the fake credential to stdout. While the pre-commit hook doesn't actually check for debug statements (only secret patterns and file size), removing it was still the right call since /pr-ready had flagged it as something that shouldn't ship.

The final file was reduced to:

bash
#!/bin/bash
echo "Notification sent"

---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 9 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)

![alt text](<screenshots/Pull-Request -showing-the-base-repo-ass6.png>)

---

#### PR Link

https://github.com/Ifeoma-Obinna23/devops-micro-internship-pravinmishra/pull/1

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**

I edited both the title and the description. The AI's original draft was written while the risky version of scripts/notify.sh was still staged, so it described the script as "intentionally contains a credential-shaped string and a debug echo" and used the title demo: add sample script with fake AWS key for git safety-net testing. By the time I opened the PR, I had already removed the fake key and the debug statement after confirming the pre-commit hook worked correctly.

---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

The PR description would have claimed the script "intentionally contains a credential-shaped string and a debug echo statement" — but by the time I opened the PR, I'd already removed both. A reviewer reading that description would believe the diff still contained a fake credential and debug output, when it actually didn't. That mismatch could cause real problems: a reviewer might waste time flagging an issue that was already fixed, or worse, might assume the description is accurate and skip actually reading the diff closely — meaning if something had still been wrong, it could slip through because the description gave false reassurance either way

---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

This assignment's files — the pre-commit hook, the /pr-ready skill definition, and the demo script used to test them — are personal coursework artifacts, not a contribution meant to be merged into the shared class repository. Opening the PR against pravinmishraaws/devops-micro-internship-pravinmishra would incorrectly propose adding this practice content to the original upstream project, which isn't the intent.

The PR exists so my own review process — fork, branch, commit, push, and open a PR — can be demonstrated and evaluated as part of the assignment. Targeting my own fork (Ifeoma-Obinna23/devops-micro-internship-pravinmishra) keeps the workflow self-contained: it proves I can execute the full collaboration loop correctly, without actually merging assignment-specific files into the shared repository everyone else builds from
---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

Two moments in this assignment represent Gather:

The pre-commit hook running staged=$(git diff --cached --name-only --diff-filter=ACM) and looping through each file with git diff --cached -- "$file" — this collects exactly what's staged before any judgment is applied.
/pr-ready running git diff --cached and git status at the start of its review — this reads the same staged changes so it has the actual facts to reason about.

---

**2. Which step(s) represent Analyze?**

Two moments represent Analyze — one fixed-rule, one AI-assisted:

The pre-commit hook's pattern matching — running the gathered diff through grep -qE 'AKIA[0-9A-Z]{16}|-----BEGIN (RSA|OPENSSH|PRIVATE) KEY-----' and checking file size against the 1MB limit. This is deterministic, rule-based analysis: the same input always produces the same output, with no interpretation involved.
/pr-ready's reasoning over the same diff — identifying the credential-shaped string, the leftover debug echo, and the fact that the change had no accompanying context, then drafting a PR title and description. This is judgment-based analysis: it requires understanding intent and meaning, not just matching a string pattern.

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

Human Act is the moment I personally took the irreversible, consequential actions in this workflow:

Manually edited scripts/notify.sh to remove the fake key and debug line
Ran git commit -m "add notification script" myself
Ran git push origin feature/ai-pr-ready myself
Opened the Pull Request myself, reading and editing the AI's draft before submitting it

A human — not Claude — must run these steps because committing, pushing, and opening a PR change shared history and notify other people. Once pushed, a commit exists on a remote server; once a PR is open, a reviewer may start acting on it. These aren't easily undone the way ignoring a suggestion is. If the AI could take these actions on its own, the last checkpoint where a person confirms the change is correct, intentional, and safe would disappear entirely.

This is also why /pr-ready's allowed-tools deliberately excludes Write, and its instructions explicitly forbid git commit, git push, and gh pr create — the boundary isn't just a polite request the AI is trusted to follow, it's structurally enforced by what tools it's even allowed to use. The AI's role stays strictly advisory; the accountability for what actually ships stays with the engineer.

---

**4. Which step is Verify?**

Verify is the moment I confirmed the fix actually worked, rather than assuming it did:

Re-running git commit after removing the fake key and debug line, and confirming it succeeded with no BLOCKED message — proof the pre-commit hook now passes on the cleaned-up file
Re-running /pr-ready after the fix, checking that it reported a clean risk report instead of flagging the credential and debug statement — proof the AI-assisted check also finds nothing wrong now
Reviewing the final PR page before considering the work done — confirming the base repository was actually my fork (not upstream), the branches were correct, and the title/description matched the real, current state of the code

This step closes the loop: Gather and Analyze produced findings, Human Act responded to them, and Verify is what confirms the response actually resolved the problem instead of just assuming the fix worked.

---


**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**

The fixed-rule hook catches known, mechanical patterns instantly and with zero ambiguity, but it's blind to anything outside its exact regex — like a leftover debug statement or a change with no explanation. The AI skill reasons about context and can catch those subtler issues, but its judgment isn't guaranteed the way a deterministic pattern match is, so relying on it alone would leave you exposed on the one thing the hook is 100% reliable at — together, they cover each other's blind spots.

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

https://www.linkedin.com/posts/ifeoma-akabueze_dmibypravinmishra-agenticai-claudecode-ugcPost-7486547167722115073-U0_Q/?

---

## Key Learnings

Add 3-5 bullet points on what you learned this week.

The fork → branch → commit → PR workflow isn't just ceremony — every step (separate remotes, feature branches, single-purpose commits) exists to protect a shared codebase from accidental damage, not to add friction.
A fixed rule and an AI-assisted check solve different problems — one is fast and 100% reliable for exactly what it's coded to catch; the other reasons about context but isn't guaranteed. You need both.
AI can advise, but shouldn't be trusted to act — building the /pr-ready skill with Bash/Read but no Write access taught me that the safest AI boundaries are structural, not just instructional.
A PR description is a claim, not just a formality — an AI-drafted description reflects the code at the moment it's written, not necessarily the code you actually ship. Reading and editing it before submitting matters.
Verification isn't optional — re-running the same checks after a fix (not just assuming the fix worked) is what actually closes the loop between "I think this is right" and "I confirmed this is right.

-
-
-

---

# Submission Instructions

- Ensure `hooks/pre-commit` and `.claude/skills/pr-ready/SKILL.md` are committed to your GitHub repository
- Add all required screenshots to your submission
- All written answers must be in your own words
- Do not use a real secret or credential anywhere in your submission — the fake key in Task 1 is intentional and must stay clearly fake
- Open your Pull Request against your own fork, not the shared upstream repository
- Push your final changes to your forked repository
- Include your PR link and LinkedIn post URL

---

## GitHub Repository URL

Paste your forked repository URL here:

https://github.com/Ifeoma-Obinna23/devops-micro-internship-pravinmishra

---

# Completion Checklist

- [ ] Branch `feature/ai-pr-ready` created with a staged file containing a fake secret and a debug statement
- [ ] `hooks/pre-commit` created and tracked in the repo (not only in `.git/hooks/`)
- [ ] `core.hooksPath` configured to point at `hooks/`
- [ ] Pre-commit hook shown blocking the risky commit
- [ ] `.claude/skills/pr-ready/SKILL.md` created with correct `allowed-tools` (no `Write`) and `disable-model-invocation: true`
- [ ] `/pr-ready` run against the risky diff and shown flagging issues
- [ ] Risky file fixed; `git commit` succeeds cleanly
- [ ] `/pr-ready` re-run showing a clean report and drafted PR title/description
- [ ] Pull Request opened using the AI draft as a starting point, with your own fork as the base repository (not upstream), PR link included
- [ ] Agentic Loop mapping (Task 7) completed in your own words
- [ ] LinkedIn post published and URL submitted
- [ ] All required screenshots added
- [ ] GitHub repository URL provided

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
