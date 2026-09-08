# Assignment 7 — AI-Assisted Azure Security Posture Audit

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will build a read-only Bash script that audits the Azure resources you deployed earlier this week — a virtual machine, a three-tier network with a Load Balancer, a Storage Account, and an Azure Database for MySQL server — for common security misconfigurations. You will connect that script to Claude Code as a reusable `/azure-audit` skill that explains findings and recommends a fix without ever running it, then fix one real finding yourself and prove the fix with a second audit run. This is the same read-only-evidence-then-human-fixes discipline from Week 3, now applied to Azure with the `az` CLI instead of Linux commands — and the cloud-agnostic counterpart to the AWS audit you built in Week 6.

---

# Task 1 — Confirm Your Resources and Create the Workspace

## Goal

Confirm your Azure CLI is authenticated and can see the VM, network, storage account, and MySQL server you built this week, then set up a workspace folder for the audit.

### Evidence

#### Screenshot 1 — `az account show` and `az vm list -d -o table` confirming your subscription and running VM (subscription ID partially blurred)

![alt text](screenshots/Az-account-show-and-az-vm-list-d-o-table-confirming-your-subscription-and-running-VM-ass7.png)

---

# Task 2 — Create Project Context and Safety Rules in CLAUDE.md

## Goal

Create a `CLAUDE.md` for this workspace that tells Claude what the audit covers and the safety rules it must follow: never run a mutating `az` command, never claim a finding without report evidence, and always let the human review and run any remediation.

### Evidence

#### Screenshot 2 — `CLAUDE.md` open in your editor showing the project overview, audit workflow, and safety rules

![alt text](screenshots/CLAUDE-md-open-in-your-editor-showing-the-project-overview-audit-workflow-and-safety-rules-ass7.png)

---

# Task 3 — Use Agentic AI to Plan the Audit Before Writing the Script

## Goal

Ask Claude Code to read `CLAUDE.md` and propose a read-only, four-check audit plan (NSG rules open to `0.0.0.0/0` on port 22 or 3389, storage account public blob access, VM disk encryption status, and Azure Database for MySQL public network access) — without creating or editing any file yet.

### Evidence

#### Screenshot 3 — Claude Code showing the four-check plan, with no files created or modified
![alt text](screenshots/Claude-Code-showing-the-four-check-plan-Check-A-ass7.png)

![alt text](screenshots/Claude-Code-showing-the-four-check-plan-B-ass7.png)

![alt text](screenshots/Claude-Code-showing-the-four-check-plan-Check-B-ass7.png)

![alt text](screenshots/Claude-Code-showing-the-four-check-plan-Check-C-ass7.png)

![alt text](screenshots/Claude-Code-showing-the-four-check-plan-Check-D-ass7.png)

![alt text](screenshots/Claude-Code-showing-the-four-check-plan-Check-D-b-ass7.png)
---

# Task 4 — Build the Azure Audit Bash Script

## Goal

Write a Bash script that runs the four checks from Task 3 using read-only `az` commands, writes a PASS/WARN/FAIL report with your Full Name, and exits with a different code for a healthy, warning, or failing result. Validate it with `bash -n` and make it executable.

### Evidence

#### Screenshot 4 — Your script open in your editor, showing the check functions and the `az` commands they call

![alt text](screenshots/Script-open-in-editor-showing-the-check-functions-ass7.png)

![alt text](screenshots/Script-open-in-your-editor-showing-the-check-functions-and-the-az-commands-ass7.png)

![alt text](screenshots/Script-open-in-editor-showing-the-check-functions-and-the-az-commandss-ass7.png)

![alt text](screenshots/Script-open-in-my-editor-showing-the-check-functions-az-commands-ass7.png)

![alt text](screenshots/Scripts-open-in-editor-showing-the-check-functions-and-the-az-commands-ass7.png)
---

#### Screenshot 5 — Output of `bash -n` (no syntax errors) and `ls -l` showing the script is executable

![alt text](screenshots/Output-of-bash-n-no-syntax-errors-and-ls-l-showing-the-script-is-executable-ass7.png)

---

# Task 5 — Run the Script and Review the Baseline Report

## Goal

Run the script against your live resources and read the report honestly, even if it shows a real finding — do not fix anything yet.

### Evidence

#### Screenshot 6 — Script output showing your Full Name and all four checks with a PASS, WARN, or FAIL result

![alt text](screenshots/Script-output-showing-Full-Name-and-all-four-checks-with-PASS-ass7.png)

---

# Task 6 — Create and Run the /azure-audit Skill

## Goal

Create a Claude Code skill restricted to read-only tools (no `Write`) that runs your script, reads the report, and explains every finding with the risk of leaving it unresolved — without ever running a remediation command itself.

### Evidence

#### Screenshot 7 — Your skill file's frontmatter showing `allowed-tools` without `Write`

![alt text](screenshots/skill-files-frontmatter-showing-allowed-tools-without-Write-ass7.png)

---

#### Screenshot 8 — `/azure-audit` output showing the baseline findings and Claude's explanation

![alt text](screenshots/Azure-audit-output-showing-the-baseline-findings-and-Claudes-explanation-ass7.png)

![alt text](screenshots/Azure-audit-output-showing-the-baseline-findings-and-Claude-explanation-ass7.png)

![alt text](screenshots/azure-audit-output-showing-the-baseline-findings-and-Claude-explanation-asss7.png)

![alt text](screenshots/Azure-audit-output-showing-the-baseline-finding-and-Claude-explanation-ass7.png)

---

# Task 7 — Fix a Real Finding and Re-Verify

## Goal

Pick one WARN or FAIL finding (or deliberately open an NSG rule to port 22 from `0.0.0.0/0` if your baseline was already clean), save that failing report, run the remediation command yourself — scoped to your own IP, not left open — and confirm the second audit run shows it resolved.

### Evidence

#### Screenshot 9 — Saved report showing the original finding before the fix

![alt text](screenshots/Saved-report-showing-the-original-finding-before-the-fix-ass7.png)

---

#### Screenshot 10 — Terminal output of the remediation command you ran yourself

![alt text](screenshots/Terminal-output-of-the-remediation-command-ass7.png)

---

#### Screenshot 11 — Second `/azure-audit` run (or report) showing the finding resolved

![alt text](screenshots/Azure-audit-run-or-report-showing-the-finding-resolved-ass7.png)

---

### Notes

Compare this assignment to the AWS audit you built in Week 6: which finding categories map to each other across the two clouds, and what stayed exactly the same about the workflow even though the `az`/`aws` commands are completely different?

### Comparison with the AWS Audit from Week 6

The Azure audit assignment follows almost the same security-audit structure as the AWS audit from Week 6, but it uses Azure resources and the `az` CLI instead of AWS resources and the `aws` CLI.

| AWS Week 6 Finding                                      | Azure Equivalent                                                                                                               |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| S3 public-access settings / public ACLs                 | Storage Account public blob access                                                                                             |
| Security Groups open to the internet on SSH port 22     | NSG rules open to `0.0.0.0/0` on SSH port 22                                                                                   |
| Security Groups open to the internet on MySQL port 3306 | NSG rules open to `0.0.0.0/0` on RDP port 3389 — both are checks for unnecessarily internet-exposed management/database access |
| RDS public accessibility                                | Azure Database for MySQL public network access                                                                                 |
| EBS volume encryption                                   | Azure VM disk encryption status                                                                                                |

The exact Azure services are different, but the security principles being checked are very similar: **unnecessary public exposure, unrestricted network access, and protection of data at rest**.

### What stayed exactly the same

The **workflow stayed the same even though the CLI commands changed completely**:

**1. Gather →** A Bash script uses read-only cloud CLI commands to collect evidence from the live environment and produces a PASS/WARN/FAIL report.

**2. Analyze →** Claude Code reads the report, explains what each finding means, assesses the security or cost risk, and recommends a remediation. Claude does not make the change itself.

**3. Human Act →** The engineer reviews the recommendation and manually runs the remediation command. The human remains the final decision-maker before any live cloud resource is changed.

**4. Verify →** The audit script is run again to confirm that the finding has actually been resolved.

The same safety principles also remained unchanged. Claude must not execute mutating commands, must not claim a finding without evidence in the audit report, and must never automatically remediate a problem.

Therefore, the main difference between the two assignments is **the cloud provider and the commands used**, not the engineering methodology. AWS uses commands such as `aws ec2 describe-security-groups` and `aws rds describe-db-instances`, while Azure uses corresponding read-only `az` commands such as `az network nsg rule list` and Azure resource queries. The underlying discipline remains:

**Read-only evidence → AI analysis → Human-controlled remediation → Verification.**

This demonstrates that the audit approach is **cloud-agnostic**: the tools and resource names change, but the security principles and human-in-the-loop workflow remain the same.


---

# Submission Instructions

Complete all tasks in sequence.

Your submission must include:
- All 11 required screenshots
- Do not expose your Azure subscription ID, tenant ID, client secrets, or connection strings

---

# Completion Checklist

- [ ] Task 1: Azure resources confirmed and workspace created (Screenshot 1)
- [ ] Task 2: `CLAUDE.md` created with project context and safety rules (Screenshot 2)
- [ ] Task 3: Claude produced a read-only four-check plan before any script existed (Screenshot 3)
- [ ] Task 4: Audit script built, syntax-checked, and executable (Screenshots 4–5)
- [ ] Task 5: Baseline audit run and reviewed honestly (Screenshot 6)
- [ ] Task 6: `/azure-audit` skill created with no `Write` permission and run successfully (Screenshots 7–8)
- [ ] Task 7: A real finding fixed by you (not Claude) and re-verified as resolved (Screenshots 9–11)
- [ ] Notes comparing this to the Week 6 AWS audit completed
- [ ] No subscription IDs, tenant IDs, or credentials exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
