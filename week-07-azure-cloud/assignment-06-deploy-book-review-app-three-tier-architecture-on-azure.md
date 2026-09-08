# Assignment 6 — Capstone: Deploy Book Review App (Three-Tier Architecture) on Azure

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

This is the most important assignment of the course. You will deploy the Book Review App in a production-ready, best-practice-compliant three-tier architecture on Azure: separated presentation, application, and database tiers, least-privilege network access, a controlled public entry point, protected secrets, and availability/monitoring evidence.

---

# Task 1 — Design the Azure Three-Tier Architecture

## Goal

Create an architecture diagram and implementation plan identifying the presentation, application, and database components, the chosen Azure services, the public entry point, and the internal traffic paths.

### Evidence

#### Screenshot 1 — Architecture diagram showing the public entry point, three tiers, network boundaries, and traffic flow

![alt text](screenshots/book-review-app-three-tier-architecture.png)

---

#### Screenshot 2 — Written architecture assumptions and selected Azure services

[text](screenshots/written-architecture-assumptions-ass6.docx)

---

# Task 2 — Create the Azure Network Foundation

## Goal

Create a dedicated Resource Group and VNet with separate subnets for the web, application, and database tiers, keeping the application and database tiers without direct public access.

### Evidence

#### Screenshot 3 — Resource Group overview showing the assignment resources

![alt text](screenshots/Resource-Group-overview-showing-the-assignment-resources-ass6.png)

---

#### Screenshot 4 — VNet overview showing the address space and all required subnets

![alt text](screenshots/VNet-overview-showing-the-address-space-and-all-required-subnets-ass6.png)

---

#### Screenshot 5 — Route-table or Private DNS evidence where applicable
![alt text](screenshots/Route-table-ass6.png)

---

# Task 3 — Configure Security and Secret Management

## Goal

Apply least-privilege NSG rules so traffic flows Internet → public entry point → web tier → application tier → database tier, and store credentials in Azure Key Vault or another approved secure mechanism.

### Evidence

#### Screenshot 6 — NSG rules proving least-privilege access between the tiers

![alt text](screenshots/NSG-rules-proving-least-privilege-access.png)

![alt text](screenshots/NSG-rules-proving-least-privilege-access-ass6.png)

![alt text](screenshots/NSG-rules-proving-least-privilege-access-between-tiers-ass6.png)
---

#### Screenshot 7 — Key Vault or approved secret-management configuration (without displaying secret values)

![alt text](<screenshots/Key Vault-or-approved-secret-management-configuration-ass6.png>)

---

# Task 4 — Deploy the Presentation (Web) Tier

## Goal

Deploy the Book Review App presentation layer on the approved web-tier compute service, configured to route requests to the internal application-tier endpoint, and not directly exposed except through the public entry service.

### Evidence

#### Screenshot 8 — Web-tier compute overview showing subnet and availability configuration
![alt text](screenshots/Web-tier-compute-overview-showing-subnet-and-availability-configuration-ass6.png)

---

#### Screenshot 9 — Terminal or service output proving the presentation layer is running

![alt text](screenshots/Terminal-or-service-output-proving-the-presentation-layer-is-running-ass6.png)

---

# Task 5 — Deploy the Business (Application) Tier

## Goal

Deploy the Book Review App backend privately in the application subnet, configured to use the private database endpoint and secured environment values, reachable only through its internal endpoint.

### Evidence

#### Screenshot 10 — Application-tier compute overview showing private subnet placement

![alt text](screenshots/Application-tier-compute-overview-showing-private-subnet-placement-ass6.png)

---

#### Screenshot 11 — Backend process, service, or listening-port evidence

![alt text](screenshots/Backend-process-service-or-listening-port-evidence.png)

---

#### Screenshot 12 — Internal health-check or API response (without exposing secrets)

![alt text](screenshots/Internal-health-check-or-API-response-ass6.png)

---

# Task 6 — Deploy the Managed Database Tier

## Goal

Create a private Azure managed database (public access disabled), with availability/backup/retention settings, the Book Review App schema imported, and access restricted to the application tier only.

### Evidence

#### Screenshot 13 — Database overview showing private connectivity and public access disabled
![alt text](screenshots/Database-overview-showing-private-connectivity-and-public-access-disabled-ass6.png)

---

#### Screenshot 14 — Availability, backup, and retention configuration

![alt text](screenshots/Availability-backup-and-retention-configuration-ass6.png)

---

#### Screenshot 15 — Successful schema or connectivity verification (without exposing credentials)
![alt text](screenshots/Successful-schema-or-connectivity-verification-ass6.png)


---

# Task 7 — Configure Traffic Management, Availability, and Monitoring

## Goal

Configure the approved public entry service with health probes and backend pools, internal routing for the application tier where required, and enable Azure Monitor/diagnostics/logs/alerts for the key resources.

### Evidence

#### Screenshot 16 — Public entry service showing listener, frontend endpoint, and healthy web targets

![alt text](screenshots/Public-entry-service-showing-listener-frontend-endpoint-ass6.png)
![alt text](screenshots/Public-entry-service-showing-listener-frontend-endpoint-and-healthy-web-targets-ass6.png)
![alt text](screenshots/Public-entry-service-showing-listener-frontend-endpoint-and-healthy-web-targets-ass6b.png)
---

#### Screenshot 17 — Internal application-tier load-balancing or routing configuration where applicable

![alt text](screenshots/Internal-application-tier-load-balancing-IP-config-ass6.png)

![alt text](<screenshots/Internal-application-tier load-balancing-backend-config-ass6.png>)

![alt text](screenshots/Internal-application-tier-load-balancing-backend-health-probe-ass6.png)
---

#### Screenshot 18 — Azure Monitor, diagnostic settings, logs, metrics, or alert evidence

![alt text](screenshots/Azure-Monitor-diagnostic-settings-metrics-ass6.png)

---

# Task 8 — Validate the Production-Style Deployment

## Goal

Confirm the Book Review App works end to end through the public endpoint, with at least one database read and one write, confirm private tiers are not internet-reachable, and complete a safe availability test.

### Evidence

#### Screenshot 19 — Browser showing the Book Review App through the public endpoint

![alt text](screenshots/Browser-showing-the-Book-Review-App-through-the-public-endpoint-ass6.png)

---

#### Screenshot 20 — Proof of successful database-backed read and write operations

![alt text](screenshots/Proof-of-successful-database-backed-read-and-write-operations-ass6.png)

---

#### Screenshot 21 — Evidence that private tiers are not publicly accessible

![alt text](screenshots/Evidence-that-private-tiers-are-not-publicly-accessible-ass6.png)

---

#### Screenshot 22 — Availability-test and healthy-target evidence

![alt text](screenshots/Availability-test-and-healthy-target-evidence-ass6.png)

---

#### Public Endpoint

Paste your public endpoint URL here:

http://20.252.13.192

---

### Notes

Summarize what worked, issues encountered and how they were fixed, and the availability/security/secrets/monitoring/backup choices made.

What worked and issues encountered

Full three-tier architecture deployed exactly as designed: Internet → Public Load Balancer → Web tier (Nginx + Next.js) → Internal Load Balancer → Application tier (Node.js/Express) → Azure Database for MySQL Flexible Server (private access only).
Both the web and application tier VMs run with no public IP; the only public entry point is the Load Balancer's frontend IP.
Database credentials, connection details, and the JWT secret are stored in Azure Key Vault and retrieved at runtime by the app-tier VM using a system-assigned Managed Identity — no secrets hardcoded anywhere.
End-to-end functionality verified through the public endpoint: homepage loads real book data, and a new user registration was confirmed to write successfully into the database.
Confirmed from outside the VNet that the app tier's private IP times out on direct connection — private tiers are not reachable from the internet.
Public Load Balancer reported 100% Data Path Availability with a consistently healthy backend.
Issues encountered and how they were fixed
Issue	Root cause	Fix
MySQL Flexible Server "Advanced Create" hidden	Default simplified wizard omits the Networking tab entirely	Used the "Advanced Create" link from the Review page to reach Private access (VNet Integration) options
Backend crashed on startup: "insecure transport prohibited"	Azure MySQL enforces SSL by default; Sequelize config had no SSL options	Added dialectOptions.ssl (require: true, rejectUnauthorized: false)
MySQL access denied despite correct password	Password began with #, treated as a comment character when unquoted in .env	Wrapped the value in double quotes
Frontend showed "No books available"	Book-fetching code is a client component; it ran in the browser and tried to call the app tier's private IP directly — unreachable from the public internet	Added an /api/ reverse-proxy block in the web tier's Nginx config, pointing to the Internal Load Balancer
Registration/login still failed after the proxy fix	Second bug in the repo's api.js: (1) `	
Registration returned HTTP 500	Backend's CORS allow-list only included localhost:3000, not the Load Balancer's public IP	Added the public IP to ALLOWED_ORIGINS and restarted PM2
Backend/web VMs had no outbound internet access	NAT Gateway was only attached to app-subnet and db-subnet, not web-subnet	Associated web-subnet with the same NAT Gateway
Availability

Web and app tiers each run as a single VM behind their respective load balancer, with health probes removing an unresponsive instance from rotation. MySQL runs on Burstable B1ms with automated backups; zone-redundant HA was left disabled to control cost. PM2 keeps both Node.js processes running, auto-restarting on crash, and is configured via systemd to relaunch on VM reboot.

Security

Three-tier network isolation via per-subnet NSGs allowing only minimum required traffic (HTTP/HTTPS into web, port 3001 into app from web-subnet only, port 3306 into db from app-subnet only). No SSH exposed publicly — admin access only via Azure Bastion. NAT Gateway provides outbound-only internet access for private subnets. MySQL deployed with Private access (VNet Integration), public access disabled.

Secrets management

Key Vault (RBAC model) holds five secrets: DB-HOST, DB-NAME, DB-USER, DB-PASS, JWT-SECRET. App-tier VM has a system-assigned Managed Identity granted Key Vault Secrets User, scoped to this vault only. Azure CLI authenticated via az login --identity retrieves secrets into the backend's local .env, never committed to source control.

Monitoring

A dedicated Log Analytics workspace collects diagnostics from the Public Load Balancer. Metrics (Data Path Availability, Health Probe Status) reviewed directly showed sustained 100% availability with a continuously healthy backend target.

Backup and recovery

MySQL Flexible Server has automated backups with 7-day retention, locally-redundant storage. Both VMs are treated as stateless/reproducible from source and configuration rather than individually backed up.

---

# Submission Instructions

- Add all required screenshots and links in your submission
- Do not expose passwords, keys, connection strings, or subscription IDs

---

# Completion Checklist

- [ ] Task 1: Architecture diagram and assumptions documented (Screenshots 1–2)
- [ ] Task 2: Network foundation created with isolated tiers (Screenshots 3–5)
- [ ] Task 3: Least-privilege security and secret management configured (Screenshots 6–7)
- [ ] Task 4: Presentation tier deployed (Screenshots 8–9)
- [ ] Task 5: Application tier deployed privately (Screenshots 10–12)
- [ ] Task 6: Managed database tier deployed privately (Screenshots 13–15)
- [ ] Task 7: Public entry, internal routing, and monitoring configured (Screenshots 16–18)
- [ ] Task 8: End-to-end validation and availability test completed (Screenshots 19–22, Public Endpoint, Notes)
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
