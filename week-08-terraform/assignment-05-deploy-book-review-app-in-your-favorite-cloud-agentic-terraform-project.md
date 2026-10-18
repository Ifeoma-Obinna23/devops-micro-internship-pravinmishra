# Capstone Assignment — Deploy the Book Review App Using Terraform and Claude Code Agentic AI

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Student Details

**Full Name:** Ifeoma Akabueze 
**Cloud Platform:** AWS
**GitHub Repository URL:** https://github.com/Ifeoma-Obinna23/devops-micro-internship-pravinmishra.git 
**Public Application URL / Load-Balancer DNS:** book-review-app-dev-public-alb-2132009396.us-east-1.elb.amazonaws.com

---

## Purpose

Deploy the Book Review App using Terraform on AWS or Azure in a secure, highly available, production-style three-tier architecture. Use Claude Code, specialized subagents, Terraform MCP, and validation hooks to support the engineering workflow while keeping all infrastructure-changing operations under human control.

---

# Task 0 — Prepare the Project and Agentic AI Environment

## Goal

Prepare the Book Review App project and configure the provided Claude Code Agentic AI starter kit with project context, specialized subagents, Terraform MCP, validation hooks, and safety guardrails.

## Evidence

### Screenshot 1 — Project `CLAUDE.md`

Add a screenshot of the project `CLAUDE.md` showing the three-tier architecture, security boundaries, Terraform requirements, and human-approval rules.

![alt text](screenshots/CLAUDEmd-showing-the-three-tier-architecture-security-boundaries-ass5.png)

![alt text](screenshots/CLAUDEmd-showing-the-three-tier-architecture-security-boundaries-terraform-requirements-ass5.png)

![alt text](screenshots/CLAUDEmd-showing-the-three-tier-architecture-security-boundaries-terraform-requirements-and-human-approval-rules-ass5.png)

![alt text](screenshots/CLAUDE-md-showing-the-three-tier-architecture-and-human-approval-rules-ass5.png)
---

### Screenshot 2 — Terraform Engineer Subagent

Add a screenshot showing the Terraform Engineer subagent configuration.

![alt text](screenshots/Terraform-Engineer-subagent-configuration-ass5.png)

---

### Screenshot 3 — Architecture and Security Reviewer Subagent

Add a screenshot showing the Architecture and Security Reviewer subagent configuration.

![alt text](screenshots/Architecture-and-Security-Reviewer-subagent-configuration-ass5.png)

![alt text](screenshots/Architecture-and-Security-Reviewer-subagent-configuration-ass5b.png)

---

### Screenshot 4 — Terraform MCP Connection

Add a screenshot showing Terraform MCP connected and available.

![alt text](screenshots/Terraform-MCP-connected-and-available-ass5.png)

---

### Screenshot 5 — Validation Hooks

Add a screenshot showing the configured Claude Code validation hooks.

![alt text](screenshots/Configured-Claude-Code-validation-hooks-ass5.png)

---

# Task 1 — Design the Three-Tier Architecture

## Goal

Design the required secure, highly available three-tier architecture and create an architecture diagram before building the infrastructure.

The diagram must show:

- VPC or VNet
- Availability Zones or equivalent availability locations
- Six subnets
- Internet connectivity
- NAT or outbound design
- Public load balancer
- Web Tier
- Internal load balancer
- Application Tier
- Managed MySQL
- Read replica
- Main traffic flow

## Architecture Diagram

![alt text](screenshots/book_review_app_three_tier_architecture-ass5.png)

---

# Task 2 — Build the Terraform Networking and Security Layers

## Goal

Create the modular Terraform project and implement the network and security layers across the required public and private subnets.

## Evidence

### Screenshot 6 — Modular Terraform Project Structure

Add a screenshot showing the modular Terraform project structure.

![alt text](screenshots/modular-Terraform-project-structure-ass5.png)

![alt text](screenshots/Modularr-Terraform-project-structure-ass5.png)

---

### Screenshot 7 — Six-Subnet Architecture

Add a screenshot showing the six-subnet architecture across two availability locations.

![alt text](screenshots/Six-subnet-architecture-across-two-availability-locations-ass5.png)

---

### Screenshot 8 — Public and Private Tier Separation

Add a screenshot showing the public and private tier separation, including routing and security boundaries.

![alt text](screenshots/public-and-private-tier-separation-including-routing-and-security-boundaries-ass5.png)

![alt text](screenshots/public-and-private-tier-separation-including-security-boundaries-ass5.png)

![alt text](screenshots/Public-and-private-tier-separation-including-routing-and-security-boundares-ass5.png)
---

# Task 3 — Build the Load-Balancing and Compute Layers

## Goal

Deploy the public and internal load balancers and the Web and Application compute resources required by the Book Review App.

## Evidence

### Screenshot 9 — Web and Application Compute

Add a screenshot showing the Web and Application compute resources in their required subnets.

![alt text](screenshots/Web-compute-resources-in-their-required-subnets-ass5.png)

![alt text](screenshots/Application-compute-resource-in-their-required-subnets-ass5.png)
---

### Screenshot 10 — Public Load Balancer

Add a screenshot showing the internet-facing public load balancer.

![alt text](screenshots/Internet-facing-public-load-balancer-ass5.png)

---

### Screenshot 11 — Internal Load Balancer

Add a screenshot showing the private internal load balancer.

![alt text](screenshots/Private-internal-load-balancer-ass5.png)

---

### Screenshot 12 — Healthy Targets

Add a screenshot showing healthy target groups or backend pools.

![alt text](screenshots/Healthy-target-groups-ass5.png)

---

# Task 4 — Build the Managed MySQL Database Layer

## Goal

Deploy a private, highly available managed MySQL database with a read replica and restrict database connectivity to the Application Tier.

## Evidence

### Screenshot 13 — Managed MySQL Database

Add a screenshot showing the managed MySQL database deployment.

![alt text](screenshots/Managed-MySQL-database-deployment-ass5.png)

---

### Screenshot 14 — High Availability

Add a screenshot showing the Multi-AZ or high-availability configuration.

![alt text](screenshots/Multi-AZ-high-configuration-ass5.png)

---

### Screenshot 15 — Read Replica

Add a screenshot showing the read replica configuration.

![alt text](screenshots/Read-replica-configuration-ass5.png)

---

### Screenshot 16 — Private Database Access

Add a screenshot showing that the database is private and accepts MySQL traffic only from the Application Tier.

![alt text](screenshots/Database-private-and-accepts-MySQL-traffic-only-from-Application-Tier-ass5.png)

![alt text](screenshots/Database-is-private-and-accepts-MySQL-traffic-only-from-the-pp-Tier-ass5.png)
---

# Task 5 — Validate, Review, and Apply the Terraform Configuration

## Goal

Validate the Terraform configuration, review the execution plan using both Agentic AI and human judgment, and apply the infrastructure changes only after all required checks pass.

## Evidence

### Screenshot 17 — Terraform Validation

Add a screenshot showing successful `terraform validate` output.

![alt text](screenshots/Successful-terraform-validate-output-ass5.png)

---

### Screenshot 18 — Terraform Plan

Add a screenshot showing the Terraform plan output.

![alt text](screenshots/Terraform-plan-output-ass5.png)

---

### Screenshot 19 — Terraform Apply

Add a screenshot showing successful `terraform apply` completion.

![alt text](screenshots/Successful-terraform-apply-completion-ass5.png)

![alt text](screenshots/successful-terraform-apply-completion-ass5a.png)

---

# Task 6 — Deploy and Configure the Book Review Application

## Goal

Deploy and configure the Book Review App across the Web, Application, and Database tiers and verify the complete application functionality.

## Evidence

### Screenshot 20 — Homepage

Add a screenshot showing the Book Review App homepage through the public endpoint.

![alt text](screenshots/Book-Review-App-homepage-through-public-endpoint-ass5.jpeg)

---

### Screenshot 21 — Login or Authentication

Add a screenshot showing successful login or authentication.

Add your screenshot here.

---

### Screenshot 22 — Book Data

Add a screenshot showing the book listing or book details.

Add your screenshot here.

---

### Screenshot 23 — Review Functionality

Add a screenshot showing the review functionality working successfully.

![alt text](screenshots/Review-Functionality-success-ass5.png)

---

### Screenshot 24 — Backend or API Evidence

Add a screenshot showing that the backend or API is working successfully.

![alt text](screenshots/Backend-working-successfully-ass5.png)

---

### Screenshot 25 — Database Reads and Writes

Add a screenshot showing successful database reads and writes.

![alt text](screenshots/successful-database-reads-and-writes-ass5.png)

## Public Application URL

**Public Application URL / DNS:** book-review-app-dev-public-alb-2132009396.us-east-1.elb.amazonaws.com

---

# Task 7 — Demonstrate the Agentic AI Workflow

## Goal

Demonstrate how Claude Code assisted with Terraform generation, architecture and security review, and evidence-based troubleshooting while infrastructure-changing decisions remained under human control.

You do not need to submit your complete Claude Code conversation history. Include only focused evidence.

## Evidence

### Screenshot 26 — AI-Assisted Terraform Generation

Add a screenshot showing one useful example of AI-assisted Terraform generation or improvement.

![alt text](screenshots/AI-assisted-Terraform-generation-ass5.png)

---

### Screenshot 27 — Architecture or Security Review

Add a screenshot showing one structured architecture or security review result.

![alt text](screenshots/Architecture-Review-ass5.png)
![alt text](screenshots/Architecture-Review-asss5.png)
![alt text](screenshots/Architecturre-Review-ass5.png)

---

### Screenshot 28 — AI-Assisted Troubleshooting

Add a screenshot showing one AI-assisted troubleshooting interaction based on collected evidence.

![alt text](screenshots/AI-assisted-troubleshooting-interaction-based-collected-evidence-ass5.png)

---

# Task 8 — Complete the Final Architecture Review

## Goal

Review the completed infrastructure against the original capstone requirements and resolve significant architecture, security, reliability, and cost issues.

Confirm that the final review covers:

- Tier separation
- Availability
- Public exposure
- Routing
- Security rules
- Load balancing
- Database privacy
- Secrets
- Terraform quality
- Module structure
- Reliability
- Obvious cost risks

Use Screenshot 27 as the focused evidence for the structured architecture or security review.

---

# Task 9 — Answer the Reflection Questions

## Goal

Reflect on the architecture, Terraform implementation, and Agentic AI workflow. Answer each question briefly in your own words.

## Architecture

### 1. Why did you separate the Web, Application, and Database tiers?

I separated the Web, Application, and Database tiers to improve security, follow the principle of least privilege, and limit the impact of a potential security breach. Each tier has a specific responsibility: the Web tier handles the user interface, the Application tier handles the business logic, and the Database tier stores application data. This separation ensures that the database is not directly exposed to the internet and can only be accessed through the Application tier.


### 2. Why is the Application Tier private?

The Application Tier is kept private because it handles the application’s business logic and communicates directly with the database. By placing it in private subnets without public IP addresses, we reduce its exposure to internet-based attacks. It can only be accessed through the internal Application Load Balancer from the Web Tier, which provides an additional layer of security.


### 3. Why is MySQL private?

MySQL is kept private because it stores sensitive application data and should not be directly accessible from the internet. Placing it in private subnets and allowing access only on port 3306 from the Application Tier ensures that only authorized application servers can connect to the database. This reduces the risk of unauthorized access, brute-force attacks, and potential data breaches.


### 4. Why are multiple Availability Zones used?

Multiple Availability Zones are used to improve the application's availability and fault tolerance. By distributing resources across different data centers, the application can continue running even if one Availability Zone experiences a failure. This helps prevent a single point of failure from bringing down the entire application.


### 5. What is the difference between Multi-AZ/high availability and a read replica?

Multi-AZ is mainly used for high availability and disaster recovery. It keeps a standby database in another Availability Zone through synchronous replication, allowing the system to automatically fail over if the primary database becomes unavailable. The standby does not normally handle application traffic.

A read replica, on the other hand, uses asynchronous replication and is designed to handle read requests. It helps reduce the workload on the primary database and improves read performance and scalability.


## Terraform

### 6. How did you divide your Terraform into modules?

I divided the Terraform configuration based on the responsibility of each part of the infrastructure. I created separate modules for the Network, Load Balancing, Compute/Application, and Database layers. The Network module manages the VPC, subnets, internet gateway, NAT gateway, routing, and security groups. The Load Balancing module handles the public and internal load balancers, target groups, and listeners. The Compute module manages the EC2 instances, user data, and scaling, while the Database module manages the RDS MySQL setup, including Multi-AZ, subnet groups, and parameter groups.

### 7. How do the modules communicate through variables and outputs?

The root module connects the different modules together. Each module exposes important resource information through outputs, such as subnet IDs and security group IDs. The root main.tf then passes these outputs as input variables to the modules that depend on them. This allows the modules to work together while remaining organized and reusable.

### 8. What did you specifically check in `terraform plan`?

I used terraform plan to review the changes Terraform was about to make before applying them. I checked the number of resources being created, modified, or destroyed, reviewed the security group rules to make sure internal ports were not openly exposed, and verified that private subnets were not accidentally configured to assign public IP addresses.

## Agentic AI

### 9. What was the purpose of `CLAUDE.md`?

The CLAUDE.md file provided Claude Code with important project instructions and guidelines. It defined the required architecture, coding standards, and security rules. It also included safety instructions, such as not running terraform apply or terraform destroy automatically, so that infrastructure changes remained under my control.

### 10. What work did the Terraform Engineer subagent perform?

The Terraform Engineer subagent helped create and improve the Terraform configuration. It generated and refined the HCL code, followed Terraform formatting standards, and made sure the resources and variables were structured according to the project requirements

### 11. What did the Architecture and Security Reviewer identify?

The Architecture and Security Reviewer checked the Terraform configuration against security and architecture best practices. It looked for issues such as overly open security group rules, including exposing ports 22 or 3306 to the internet, missing encryption, and improper network or subnet separation.

### 12. Why did you use Terraform MCP instead of relying only on Claude's existing Terraform knowledge?

I used Terraform MCP to give Claude access to more current Terraform provider information and documentation. This helped reduce the risk of using outdated syntax, deprecated arguments, or resource configurations that may have changed between provider versions.

### 13. What was the purpose of your validation hooks?

The validation hooks provided automated checks before changes were accepted into the project. For example, they could run commands such as terraform fmt -check and syntax validation. This helped catch formatting or configuration errors early and prevented non-compliant code from being introduced.

### 14. Describe one real issue Claude helped you troubleshoot.

Claude helped me troubleshoot an issue with the Application Load Balancer health check. The target group was reporting the backend as unhealthy because the health check path configured on the ALB did not match the Node.js API's actual health endpoint, which was /health. Identifying and correcting that mismatch allowed the health check to work properly.

### 15. Describe one recommendation you reviewed, modified, or rejected instead of accepting blindly.

Claude suggested placing the NAT Gateway in a private subnet to keep it secure. I reviewed the recommendation against how AWS networking works and rejected it because a NAT Gateway needs to be placed in a public subnet so it can communicate with the Internet Gateway. The private subnets then route their outbound internet traffic through the NAT Gateway.

---

# Task 10 — Publish the Mandatory LinkedIn Post

## Goal

Publish a LinkedIn post describing the capstone, the technical work completed, the Agentic AI workflow, and the lessons learned.

Write the post in your own words, include at least one project image or other proof, and ensure that it can be viewed by the submission reviewer.

## LinkedIn Post URL

**LinkedIn Post URL:** https://www.linkedin.com/posts/ifeoma-akabueze_just-wrapped-up-a-capstone-project-that-pushed-activity-7511362236444094464-809r

---

# Submission Instructions

- Complete Tasks 0–10 in sequence.
- Include all Screenshots 1–28 exactly as specified.
- Ensure that your full name is visible in the required screenshots.
- Include the selected cloud platform.
- Include the completed architecture diagram.
- Include the modular Terraform project structure.
- Include the working public application URL or public load-balancer DNS.
- Include all required Agentic AI workflow evidence.
- Answer all 15 reflection questions briefly in your own words.
- Include the published LinkedIn post URL.
- Do not expose cloud credentials, database passwords, SSH private keys, JWT secrets, access tokens, account IDs, Terraform state containing sensitive values, or other confidential information.
- Review all screenshots and project files carefully before submitting through GitHub.

---

# Completion Checklist

- [ ] Selected AWS or Azure
- [ ] Added and reviewed the Agentic AI starter files
- [ ] Configured `CLAUDE.md`
- [ ] Configured the Terraform Engineer subagent
- [ ] Configured the Architecture and Security Reviewer subagent
- [ ] Connected Terraform MCP
- [ ] Configured validation hooks and safety guardrails
- [ ] Created the architecture diagram
- [ ] Created the six-subnet design
- [ ] Configured public Web Tier routing
- [ ] Kept the Application Tier private
- [ ] Kept the Database Tier private
- [ ] Configured tier-specific Security Groups or NSGs
- [ ] Restricted backend port `3001`
- [ ] Restricted MySQL port `3306` to the Application Tier
- [ ] Created the public load balancer
- [ ] Created the internal load balancer
- [ ] Configured listeners and health checks
- [ ] Deployed the Web Tier compute resources
- [ ] Deployed the private Application Tier compute resources
- [ ] Provisioned private managed MySQL
- [ ] Configured Multi-AZ or high availability
- [ ] Configured a read replica
- [ ] Created the modular Terraform project
- [ ] Used variables, outputs, and module dependencies
- [ ] Used current Terraform documentation through MCP
- [ ] Used hooks for deterministic validation
- [ ] Completed `terraform fmt`
- [ ] Completed `terraform validate`
- [ ] Reviewed `terraform plan`
- [ ] Completed the Terraform Engineer review
- [ ] Completed the Architecture and Security review
- [ ] Applied the infrastructure only after human approval
- [ ] Deployed and configured the backend
- [ ] Deployed and configured the frontend
- [ ] Configured Nginx where required
- [ ] Configured the internal backend endpoint
- [ ] Configured the public frontend endpoint
- [ ] Verified the homepage
- [ ] Verified login or authentication
- [ ] Verified book data
- [ ] Verified review functionality
- [ ] Verified the backend API
- [ ] Verified database reads and writes
- [ ] Verified healthy load-balancer targets
- [ ] Included AI-assisted Terraform generation evidence
- [ ] Included one architecture or security review
- [ ] Included one AI-assisted troubleshooting example
- [ ] Completed the final architecture review
- [ ] Answered all 15 reflection questions
- [ ] Published the mandatory LinkedIn post
- [ ] Added the LinkedIn post URL
- [ ] Captured all 28 required screenshots
- [ ] Confirmed that my full name is visible in the required screenshots
- [ ] Checked that no secrets or sensitive information are exposed

---

## About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory), focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations through hands-on experience.

---

## Resources

- Book Review App Repository: [https://github.com/pravinmishraaws/book-review-app](https://github.com/pravinmishraaws/book-review-app)
- DMI Official Website: [https://dmi.pravinmishra.com](https://dmi.pravinmishra.com)
- University: [https://university.pravinmishra.com](https://university.pravinmishra.com)
- Discord Community: [https://discord.pravinmishra.com](https://discord.pravinmishra.com)
- Blog: [https://dmi.pravinmishra.com/blog](https://dmi.pravinmishra.com/blog)
- YouTube Playlist: [https://www.youtube.com/playlist?list=PLFeSNDtI4Cho](https://www.youtube.com/playlist?list=PLFeSNDtI4Cho)
- Pravin Mishra on LinkedIn: [https://www.linkedin.com/in/pravin-mishra-aws-trainer/](https://www.linkedin.com/in/pravin-mishra-aws-trainer/)
- CloudAdvisory on LinkedIn: [https://www.linkedin.com/company/thecloudadvisory/](https://www.linkedin.com/company/thecloudadvisory/)

---

*This submission is part of the DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
