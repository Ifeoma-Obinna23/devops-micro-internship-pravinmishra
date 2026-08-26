# Assignment 5 — Deploy a Highly Available Two-Tier Application on AWS (VPC + ALB + ASG + Multi-AZ RDS)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will design and deploy a highly available two-tier web application on AWS: highly available networking across two Availability Zones, an Application Load Balancer, an Auto Scaling Group for the web tier, and a private Multi-AZ RDS database. You must prove high availability with real failure tests.

---

# Task 1 — Create HA Networking (VPC + 4 Subnets + IGW + NAT + Route Tables)

## Goal

Build a VPC (10.0.0.0/16) with two public and two private subnets across two Availability Zones, an Internet Gateway, a NAT Gateway, and the matching public/private route tables.

### Evidence

#### Screenshot 1 — VPC details showing CIDR 10.0.0.0/16

![alt text](screenshots/VPC-details-showing-CIDR-ass5.png)

---

#### Screenshot 2 — Subnets list showing four subnets and their Availability Zones

![alt text](screenshots/Subnets-list-showing-four-subnets-ass5.png)

---

#### Screenshot 3 — Public route table showing the Internet Gateway route and both public-subnet associations

![alt text](screenshots/Public-route-table-showing-Internet-Gateway-route-and-both-public-subnet-associations-ass5.png)

---

#### Screenshot 4 — Private route table showing the NAT Gateway route and both private-subnet associations

![alt text](screenshots/Private-route-table-showing-NAT-Gateway-route-ass5.png)

---

#### Screenshot 5 — NAT Gateway status showing Available and the Elastic IP

![alt text](screenshots/NAT-Gateway-status-showing-Available-Elastic-IP.png)

---

# Task 2 — Create Security Groups (ALB, EC2, RDS) with Least Privilege

## Goal

Create `ha-alb-sg` (HTTP public), `ha-web-sg` (HTTP only from `ha-alb-sg`, SSH from your IP), and `ha-db-sg` (database port only from `ha-web-sg`).

### Evidence

#### Screenshot 6 — ALB Security Group inbound rules

![alt text](screenshots/ALB-Security-Group-inbound-rules.png)

---

#### Screenshot 7 — EC2 Security Group inbound rules showing the ALB Security Group reference and SSH from your IP

![alt text](screenshots/EC2-SG-inbound-rules-showing-ALB-Security-Group-reference.png)

---

#### Screenshot 8 — RDS Security Group inbound rule showing the database port allowed only from the EC2 Security Group

![alt text](screenshots/RDS-SG-inbound-rule-showing-database-port-allowed-only-from-EC2-SG-ass5.png)

---

# Task 3 — Deploy Database Tier (RDS Multi-AZ in Private Subnets)

## Goal

Launch a private, Multi-AZ RDS database (MySQL or PostgreSQL) using the private DB Subnet Group and `ha-db-sg`.

### Evidence

#### Screenshot 9 — RDS summary showing Multi-AZ = Yes and Publicly accessible = No

![alt text](screenshots/RDS-summary-showing-Multi-AZ-Yes-ass5.png)

---

#### Screenshot 10 — RDS connectivity section showing the DB Subnet Group and Security Group

![alt text](screenshots/RDS-connectivity-section-showing-DB-Subnet-Group-ass5.png)

![alt text](screenshots/RDS-connectivity-section-showing-DB-Security-Group-ass5.png)
---

# Task 4 — Build a Launch Template (User Data Installs App + Connects to DB)

## Goal

Create a Launch Template whose user data installs the web-server runtime, deploys the application, configures the database connection, and starts the required services.

### Evidence

#### Screenshot 11 — Launch Template details showing that user data exists, including a visible snippet

![alt text](screenshots/Launch-Template-details-showing-that-user-data-exists-ass5.png)

---

#### Screenshot 12 — A running instance created from the template showing that the application responds on port 80 through a local test or browser using its public IP

![alt text](screenshots/Running-instance-from-template-showing-the-app-responds-on-port80-using-publicIP-ass5.png)

---

# Task 5 — Create an Application Load Balancer (ALB) Across 2 Public Subnets

## Goal

Create an internet-facing ALB across both public subnets with an HTTP listener and a healthy instance target group.

### Evidence

#### Screenshot 13 — ALB details showing two public subnets in two Availability Zones

![alt text](screenshots/ALB-details-showing-two-public-subnets-in-two-AZ-ass5.png)

---

#### Screenshot 14 — Target group showing at least one healthy target

![alt text](screenshots/Target-group-showing-at-least-one-healthy-target-ass5.png)

---

# Task 6 — Create Auto Scaling Group (ASG) in 2 Public Subnets

## Goal

Create an Auto Scaling Group from the Launch Template across both public subnets, with desired capacity 2, minimum 2, and maximum 4, registered to the ALB target group.

### Evidence

#### Screenshot 15 — Auto Scaling Group showing desired, minimum, and maximum capacity and the selected subnet Availability Zones

![alt text](screenshots/ASG-showing-desired-min-and-max-capacity-and-selected-subnet-AZ-ass5.png)

---

#### Screenshot 16 — EC2 instances list showing two running instances in different Availability Zones

![alt text](screenshots/EC2-instances-list-showing-two-running-instances-in-different-AZ-ass5.png)

---

# Task 7 — Configure App to Use RDS + Validate Read/Write

## Goal

Confirm the application communicates with the RDS database through the ALB DNS name with at least one read and one write operation.

### Evidence

#### Screenshot 17 — Browser showing the application loaded through the ALB DNS name with the URL visible

![alt text](screenshots/Browser-showing-application-loaded-through-ALB-DNS-name-with-URL-visible-ass5.png)

---

#### Screenshot 18 — Proof of a database write through a UI message or database query output

![alt text](screenshots/Proof-of-database-write-through-UI-message-ass5.png)
![alt text](screenshots/database-query-output-ass5.png)
---

# Task 8 — High Availability Tests (Must Do Both)

## Goal

Test A: terminate one web instance and confirm the Auto Scaling Group replaces it automatically without interrupting the ALB.

Test B: simulate an Availability Zone impact (stop, detach, or reduce desired capacity in one AZ) and confirm the application stays available.

### Evidence

#### Screenshot 19 — EC2 showing the terminated instance and the newly launched instance; timestamps are helpful

![alt text](screenshots/Newly-launched-instance-ass5.png)
![alt text](screenshots/EC2-showing-the-terminated-instance-and-newly-launched-instance-ass5.png)
---

#### Screenshot 20 — Target group showing healthy targets after replacement

![alt text](screenshots/Target-group-showing-healthy-targets-after-replacement-ass5.png)

---

#### Screenshot 21 — Evidence that an instance was removed, detached, placed in Standby, or stopped in one Availability Zone

![alt text](screenshots/Evidence-that-an-instance-was-stopped-in-one-AZ-ass5.png)

---

#### Screenshot 22 — Browser showing that the ALB DNS endpoint still works during the change

![alt text](screenshots/Browser-showing-that-ALB-DNS-endpoint-still-works-during-change-ass5.png)

---

# Task 9 — Architecture and Test-Results Summary

## Goal

Summarize the VPC/subnet layout, the ALB and Auto Scaling Group setup, the private Multi-AZ RDS setup, and the results of both high-availability tests.

### Evidence

#### Screenshot 23 — A simple architecture diagram, which may be hand-drawn, or an AWS console overview showing the components

![alt text](screenshots/Highly-available-two-tier-architecture-ass5.png)

---

### Notes

Summarize the VPC and subnets across the two Availability Zones.

VPC, Subnets and Networking in two Availability Zones

Built a VPC (10.0.0.0/16) spanning two Availability Zones (eu-west-2a, eu-west-2b), with four subnets: two public (10.0.1.0/24, 10.0.2.0/24) hosting the ALB and web tier, and two private (10.0.11.0/24, 10.0.12.0/24) hosting the database. An Internet Gateway routes public subnet traffic; a NAT Gateway with an Elastic IP lets private subnets reach the internet for updates without being publicly reachable. Two route tables enforce this split, each explicitly associated with the correct pair of subnets. Security groups form a least-privilege chain: Internet → ALB (HTTP 80) → EC2 (HTTP from ALB only + SSH from admin IP) → RDS (MySQL 3306 from EC2 only).

Summarize the ALB and Auto Scaling Group setup.

ALB and Auto Scaling Group

An internet-facing Application Load Balancer (ha-web-alb) spans both public subnets and listens on HTTP:80, forwarding to a target group (ha-web-tg) with a / health check. A Launch Template (ha-web-lt) defines self-configuring Ubuntu instances (t3.micro) that install Apache/PHP/WordPress and connect to RDS via user data on boot. An Auto Scaling Group (ha-web-asg) maintains 2 instances (min 2, max 4) spread across both AZs, registered to the target group with ELB health checks enabled.

Summarize the private Multi-AZ RDS setup.

Private Multi-AZ RDS
A private MySQL RDS instance (ha-app-db, db.t3.micro) runs in Multi-AZ mode with a standby in the second AZ, placed in the private DB subnet group, with no public access, secured by ha-db-sg. The WordPress application performs both reads (homepage content) and writes (published posts) against this database, verified through the ALB URL.

Summarize the results of both high-availability tests.

Result of both High Availability Test

Test A (instance failure): terminated one web instance; the ASG detected the capacity gap and launched a replacement within 2–3 minutes. The ALB continued serving traffic without interruption via the surviving instance throughout, and the target group returned to 3/3 healthy once the replacement passed its checks.
Test B (AZ impact simulation): stopped one instance to simulate an AZ outage. The ASG's health checks detected it as unhealthy and proactively replaced it — a stronger result than the minimum requirement. Throughout the process, the ALB DNS endpoint remained fully accessible via the surviving Availability Zone, confirming the architecture tolerates a full AZ loss.

---

# LinkedIn Post (Required)

## Goal

Publish a LinkedIn post about the high-availability build, including the ALB URL (or a redacted screenshot), three to five lines on what you built and how you tested high availability, and one proof screenshot.

## Evidence

#### LinkedIn Post URL

https://lnkd.in/p/e9wpyX7S

https://medium.com/@ifeomaohachosim/building-a-highly-available-two-tier-application-on-aws-02cc0a49c47b?sharedUserId=ifeomaohachosim

---

#### Screenshot of LinkedIn post

![alt text](screenshots/Screenshot-of-LinkedIn-post-ass5.png)

---

# Submission Instructions

- Add all required screenshots in your submission
- Do not expose passwords, connection strings, private keys, or account IDs

---

# Completion Checklist

- [ ] Task 1: VPC, four subnets, IGW, NAT Gateway, and route tables created (Screenshots 1–5)
- [ ] Task 2: Least-privilege ALB, EC2, and RDS security groups created (Screenshots 6–8)
- [ ] Task 3: Private Multi-AZ RDS created (Screenshots 9–10)
- [ ] Task 4: Self-configuring Launch Template created and tested (Screenshots 11–12)
- [ ] Task 5: ALB created across both public subnets (Screenshots 13–14)
- [ ] Task 6: Auto Scaling Group running two instances across two AZs (Screenshots 15–16)
- [ ] Task 7: Application verified through the ALB with a database read and write (Screenshots 17–18)
- [ ] Task 8: Both high-availability tests completed (Screenshots 19–22)
- [ ] Task 9: Architecture and test-results summary completed (Screenshot 23 & Notes)
- [ ] LinkedIn post published and URL submitted
- [ ] No sensitive data exposed

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