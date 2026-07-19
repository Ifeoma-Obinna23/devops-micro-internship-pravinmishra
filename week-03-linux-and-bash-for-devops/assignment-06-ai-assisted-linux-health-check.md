# Assignment 6 — Build an AI-Assisted Linux Health Check (AI-Assisted Linux Incident Triage)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will build a read-only Bash triage script that checks the health of your Ubuntu server and Nginx application, connect it to Claude Code as a reusable `/linux-triage` skill, simulate a controlled Nginx incident, use the skill to gather and analyze evidence, recover the service manually, and verify recovery. The workflow follows the Agentic Loop: Gather → Analyze → Human Act → Verify.

---

# Task 1 — Confirm the Healthy Baseline and Create the Workspace

## Goal

Confirm that Nginx and the React application are healthy before building the automation.

### Evidence

#### Screenshot 1 — Output of `systemctl is-active nginx`, `ss -ltn | grep ':80'`, and `curl -I http://localhost`

![alt text](screenshots/systemctl-active-ass6.png)
![alt text](screenshots/curl-l-200-OK-ass6.png)

---

#### Screenshot 2 — Output of `pwd` and `find . -maxdepth 4 -type d | sort` showing the workspace folder structure

![alt text](screenshots/find-maxdepth-ass6.png)
![alt text](screenshots/pwd-ass5.png)

---

### Notes

Answer the following in your own words:

**1. What proves that Nginx is running?**

 Service status using the command
systemctl is-active nginx and it returns active

systemctl status nginx
Confirm it's actually listening on a port

sudo ss -tulpn | grep nginx
This shows Nginx is bound to port 80 (or 443), proving it's ready to accept connections, not just "running" in the abstract.

Test it locally on the server itself
bashcurl -I http://localhost
A HTTP/1.1 200 OK response is strong proof the web server is actually serving content, not just alive as a process..

---

**2. What proves that the server is listening for HTTP traffic?**

systemctl status nginx
Confirm it's actually listening on a port

---

**3. Why must you capture a healthy baseline before simulating an incident?**

One should or must capture a healthy baseline before simulating an incident. If you simulate an incident (kill a process, fill up disk space, spike CPU, etc.) without first capturing what the system looked like before, you have no reference point. Every observation during the incident becomes a guess: "is this metric high because of the incident, or was it always like this?".
 Prevents chasing pre-existing issues
Sometimes a system already has a lingering problem (a warning in logs, a slow response time) that has nothing to do with the incident you're about to simulate. Without a baseline, you might waste time investigating something that was already there, mistakenly attributing it to your test.
Makes the results measurable, not just descriptive
"Response time increased" means little on its own. "Response time went from 50ms (baseline) to 4000ms (during incident)" is concrete, provable, and demonstrates the actual severity of what you simulated.


---

# Task 2 — Create Project Context and Safety Rules in CLAUDE.md

## Goal

Tell Claude exactly what this project does and what it is not allowed to do.

### Evidence

#### Screenshot 3 — CLAUDE.md open in VS Code showing all four sections (Project Overview, Incident Workflow, Safety Rules, Output Rules)

![alt text](screenshots/CLAUDE.md-ass6.png)

---

### Notes

Answer the following in your own words:

**1. Why should Claude receive project-specific operational rules?**

With explicit operational rules, Claude's role shifts from "autonomous fixer" to "evidence-based advisor":

Gather evidence → analyze it → recommend a fix → let the human decide

This is safer specifically because a human stays in control of anything that changes the system, while still getting the benefit of Claude's analysis speed.

---

**2. Why is the human required to execute the recovery command?**

Claude can analyze evidence and suggest what it believes is the right fix, but it can still be wrong — misreading a report, missing context about the environment, or recommending a command that's technically correct but has side effects it doesn't know about (like restarting Nginx during a moment when someone's mid-transaction, or on a server that's shared with other people/projects).
A human, who understands the broader context the AI doesn't have, needs to be the one who reviews that recommendation and decides whether it's actually safe to run right now, in this specific situation.

---




**3. Which rule prevents Claude from making an unsupported diagnosis?**

# Safety Rule: 
Do not claim a root cause unless the report contains supporting evidence

---

# Task 3 — Use Agentic AI to Plan Before Writing the Script

## Goal

Use Claude Code to inspect the environment and produce a read-only plan before creating any Bash code.

### Evidence

#### Screenshot 4 — Claude Code showing the five-check plan and read-only inspection results

![alt text](screenshots/Claude.Readonly-ass6.png)
![alt text](screenshots/claude.Readonly2-ass6.png)

---

### Notes

Answer the following in your own words:

**1. Which part of this task represents the Gather phase?**

1. Nginx service status
systemctl is-active nginx
- Healthy: prints active, exit code 0
- Failed: prints inactive, failed, or Nginx is not running

2. Port 80 listening state
ss -ltn | grep ':80 '
- Healthy: a line showing LISTEN with local address *:80 or 0.0.0.0:80
- Failed: no output — nothing is bound to accept HTTP connections even if Nginx claims to be active

3. Localhost HTTP response
curl -I http://localhost
- Healthy: first line HTTP/1.1 200 OK
- Failed: connection refused/timed out, or a 5xx status — the server process isn't answering requests even if the OS-level port check passes

4. Root disk usage
df -h /
- Healthy: Use% comfortably below a warning threshold (e.g. under ~80%)
- Failed/Warn: Use% at or above threshold — a full disk can block Nginx from writing logs, PID files, or serving app assets

5. Available memory
free -h
- Healthy: available column shows a healthy amount (e.g. well above ~10–15% free)
- Failed/Warn: available very low and/or swap heavily used — memory pressure can cause Nginx or the app to be OOM-killed

---

**2. Did Claude follow the instruction not to create files? How did you verify this?**

Yes, Claude followed the instruction. Here's how you can verify it from the output itself:
Evidence in the transcript
The explicit closing statement
No files were created or edited.
Claude states this directly at the end — a clear self-report confirming compliance.

---

**3. Why is planning before coding useful in DevOps automation?**

Planning before coding is useful because it lets you catch mistakes, missing context, and risky assumptions before they can cause actual harm to a live system — which is a much higher-stakes situation than a typical coding bug.

---

# Task 4 — Build the Linux Triage Bash Script

## Goal

Create one Bash script that gathers consistent Linux and Nginx health evidence.

### Evidence

#### Screenshot 5 — Top section of `linux-triage.sh` showing variables, thresholds, and the checks array


![alt text](screenshots/linux-triage-ass6.png)
---

#### Screenshot 6 — Middle section showing check functions and conditionals

![alt text](screenshots/middle-triage-function-ass6.png)

---

#### Screenshot 7 — Bottom section showing the loop, summary function, and exit behavior

![alt text](screenshots/bottom-section-triage-ass6.png)

---

#### Screenshot 8 — Output of `bash -n scripts/linux-triage.sh` (no syntax errors) and `ls -l scripts/linux-triage.sh` showing executable permission

![alt text](screenshots/linux-triage-sh-ass6.png)
---

### Notes

Answer the following in your own words:

**1. What is stored in the checks array?**

The checks array stores the names of five functions 

---

**2. How does the `for` loop use that array?**

The for loop takes each function name stored in the checks array and runs it as an actual command, one at a time — this is what actually executes all five health checks in your script

---

**3. Why are the health checks separated into functions?**

Separating each health check into its own function keeps the script organized, reusable, and easy to modify — the same core benefits of functions.

---

**4. What is the purpose of `$(...)` in this script?**

$(...) is called command substitution — it runs the command inside the parentheses, and replaces $(...) with whatever that command outputs (its printed result), so the output can be stored in a variable or used inline.
---

**5. Why does the script use different exit codes for HEALTHY, WARN, and FAIL?**

Different exit codes let anything calling this script — another script, a cron job, a CI/CD pipeline, or a human checking $? — know the outcome without having to read or parse the actual report text. The exit code alone tells you the severity of the result.

---

# Task 5 — Run and Understand the Healthy-State Report

## Goal

Run the Bash script against the healthy server and verify that it creates a report.

### Evidence

#### Screenshot 9 — Output of `./scripts/linux-triage.sh` showing your Full Name and all five check results

![alt text](screenshots/output-linux-triage-ass6.png)

---

#### Screenshot 10 — Output showing the captured exit code and final summary

![alt text](screenshots/captured-exit-code-ass6.png)
![alt text](screenshots/final-summary-ass6.png)

---

### Notes

Answer the following in your own words:

**1. What is the overall status of your healthy baseline?**

The overall status is HEALTHY, confirmed by multiple consistent signals in the report.
The direct answer
Summary:
PASS: 5
WARN: 0
FAIL: 0
Overall Status: HEALTHY
Script Exit Code: 0

---

**2. Which exact Linux evidence proves the application is serving traffic?**

The exact line is:
[PASS] Local HTTP check returned status 200

---

**3. Did your script return exit code 0 or 1? Explain why.**

My script returned exit code 0, confirmed directly in your terminal output:
Captured Exit Code: 0
and echoed again inside the report itself:
Script Exit Code: 0

---

**4. What is the difference between a warning and a failure in this script?**

The difference comes down to severity — a warning means something is trending toward a problem but isn't broken yet, while a failure means something is actually broken right now. The script encodes this distinction through separate thresholds, separate counters, and separate exit codes.

---

# Task 6 — Create and Run the /linux-triage Skill

## Goal

Turn the Bash script into a reusable, manually invoked Agentic AI workflow.

### Evidence

#### Screenshot 11 — `SKILL.md` showing the frontmatter, allowed tool restrictions, and safety rules

![alt text](screenshots/SKILL.md-ass6.png)

---

#### Screenshot 12 — `/linux-triage` output for the healthy server

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. Why does this skill have Bash, Read, and Grep, but not Write?**

The skill only needs to run the script and read/search the resulting report; leaving out Write makes it technically impossible for the skill to modify or create files, enforcing the read-only rule at the permission level.

---

**2. Why is `disable-model-invocation: true` useful for this skill?**

It stops Claude from deciding to run this skill on its own; it only runs when you explicitly type /linux-triage, keeping a human in control of when the investigation happens.

---

**3. What part is performed by Bash, and what part is performed by Claude?**

Bash does the Gather phase (running commands, writing raw evidence to the report). Claude does the Analyze phase (reading the report, interpreting it, identifying a likely cause, suggesting a recovery step).

---

**4. Why is this better than asking Claude "Is my server healthy?" without giving it evidence?**

Without evidence, Claude would be guessing from general knowledge with no real visibility into your server; with the Bash report as grounding, every claim is tied to actual current data instead of assumption.

---

# Task 7 — Simulate an Nginx Incident and Let the Skill Diagnose It

## Goal

Create a controlled service failure, gather evidence through Bash, and let Claude analyze the evidence without taking recovery action.

### Evidence

#### Screenshot 13 — Output showing Nginx is inactive and the HTTP request fails

![alt text](screenshots/NGINX-inactive-ass6.png)

---

#### Screenshot 14 — `/linux-triage` output showing failed evidence, most likely cause, and a suggested recovery command

![alt text](screenshots/linux-triagr-ass6.png)
---

#### Screenshot 15 — `incident-failure-report.txt` showing the failed checks and your Full Name

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. Which three checks failed?**

Add your answer here.

---

**2. What evidence supports the conclusion that Nginx is unavailable?**

Add your answer here.

---

**3. Did Claude execute the recovery command? Why is that important?**

Add your answer here.

---

**4. Which phase of the Agentic Loop is represented by the Bash report?**

Add your answer here.

---

**5. Which phase is represented by Claude's explanation?**

Add your answer here.

---

# Task 8 — Recover Manually, Verify Again, and Write the Incident Summary

## Goal

Recover the service as the human operator and prove that the system is healthy again.

### Evidence

#### Screenshot 16 — Output showing Nginx is active and `curl -I http://localhost` returns 200 OK

Add your screenshot here.

---

#### Screenshot 17 — Second `/linux-triage` output showing successful recovery with no FAIL results

Add your screenshot here.

---

#### Screenshot 18 — Output of `ls -lah reports` showing both `incident-failure-report.txt` and `recovery-report.txt`

Add your screenshot here.

---

#### Screenshot 19 — `incident-summary.md` showing all required sections and your Full Name

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. What action did you execute manually?**

Add your answer here.

---

**2. What evidence proves that the service recovered?**

Add your answer here.

---

**3. Why is the second triage run necessary?**

Add your answer here.

---

**4. What could go wrong if an AI agent automatically restarted every failed service?**

Add your answer here.

---

**5. In one sentence, explain the difference between using AI as a chatbot and using AI in this agentic workflow.**

Add your answer here.

---

# Incident Summary

Fill in all seven sections below in your own words.

**Full Name:** Add your full name here

**Date:** DD/MM/YYYY

---

**1. Reported Symptom**

Add your answer here.

---

**2. Evidence Collected**

Add your answer here.

---

**3. Most Likely Cause**

Add your answer here.

---

**4. Human-Approved Recovery Action**

Add your answer here.

---

**5. Verification**

Add your answer here.

---

**6. Safety Decision**

Add your answer here.

---

**7. Agentic Loop Mapping**

Add your answer here.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`Add your URL here`

---

#### Screenshot — Published LinkedIn post

Add your screenshot here.

---

# GitHub Repository URL

Paste the URL of your GitHub folder or repository containing the assignment files here:

`Add your URL here`

---

# Submission Instructions

- Add all required screenshots in your submission
- Full Name must be visible in required screenshots and the Bash report
- All written answers must be in your own words
- Do not expose sensitive information (keys, passwords, AWS account IDs, tokens)
- GitHub URL must be included in this document

---

# Completion Checklist

- [ ] Task 1: Healthy baseline confirmed, workspace created (Screenshots 1–2, Notes answered)
- [ ] Task 2: CLAUDE.md created with all four sections (Screenshot 3, Notes answered)
- [ ] Task 3: Five-check plan produced by Claude using read-only tools (Screenshot 4, Notes answered)
- [ ] Task 4: `linux-triage.sh` created, syntax validated, executable permission set (Screenshots 5–8, Notes answered)
- [ ] Task 5: Healthy-state report generated with no FAIL result (Screenshots 9–10, Notes answered)
- [ ] Task 6: `/linux-triage` skill created and run successfully on healthy server (Screenshots 11–12, Notes answered)
- [ ] Task 7: Nginx incident simulated, failed evidence captured, Claude did not execute recovery (Screenshots 13–15, Notes answered)
- [ ] Task 8: Nginx recovered manually, recovery verified, reports saved, incident summary complete (Screenshots 16–19, Notes answered)
- [ ] Incident summary contains all seven required sections
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots and the Bash report
- [ ] Skill does not have Write permission
- [ ] Skill did not execute any recovery commands
- [ ] No sensitive data exposed

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