# Deliverable 2: Corrected Architecture Diagram

## ConnectSphere — Set 1: Professional Event Networking MVP

---

## Original Architecture Flaws Identified

The original ConnectSphere User Flow (Set 1) architecture diagram contained the following critical security vulnerabilities:

```
Original Flawed Architecture (Set 1):
======================================

User's Browser/       HTTP (Unencrypted)        Load Balancer
Mobile App      ──────────────────────────►  ┌─────────────┐
                                             │             │
                                             └──────┬──────┘
                                                    │
                                        ┌───────────┴───────────┐
                                        ▼                       ▼
                                 ┌─────────────┐        ┌─────────────┐
                                 │ Web Server   │        │ Web Server   │
                                 │ (Public EC2) │        │ (Public EC2) │
                                 └──────┬──────┘        └─────────────┘
                                        │
          ┌────────────┐         ┌──────▼──────┐        ┌──────────────┐
          │ SMS Gateway│         │ API Gateway │        │ User Database│
          │    API     │◄────────┤             ├───────►│ (No encryption│
          │ (No Auth)  │         └──────┬──────┘        │  at rest)     │
          └────────────┘                │               └──────────────┘
                                 ┌──────▼──────┐
          ┌────────────┐         │ Application │
          │   Phone    │         │   Backend   │
          │Verification│         └──────┬──────┘
          │  Service   │                │
          └────────────┘         ┌──────▼──────┐
                                 │  SQL Queries │        ┌──────────────┐
                                 │  No Prepared │        │Admin Dashboard│
                                 │  Statements  │        │ (Public,     │
                                 └──────┬──────┘        │  No Auth)    │
                                        │               └──────────────┘
                                 ┌──────▼──────┐
                                 │User Database│
                                 │  MySQL DB   │
                                 └─────────────┘
```

---

## Corrected Architecture Diagram

```
Corrected Architecture (Set 1):
================================

User's Browser/     HTTPS (TLS 1.2+)          Load Balancer
Mobile App      ══════════════════════════►  ┌──────────────┐
                  [SSL/TLS Encrypted]        │  + SSL       │
                                             │  Termination │
                                             │  + HSTS      │
                                             └──────┬───────┘
                                                    │ (Internal TLS)
                                        ┌───────────┴───────────┐
                                        ▼                       ▼
                                 ┌──────────────┐       ┌──────────────┐
                                 │ Web Server   │       │ Web Server   │
                                 │ (Private     │       │ (Private     │
                                 │  Subnet EC2) │       │  Subnet EC2) │
                                 └──────┬───────┘       └──────────────┘
                                        │
          ┌─────────────┐        ┌──────▼──────┐        ┌──────────────┐
          │ SMS Gateway │        │ API Gateway │        │ User Database│
          │    API      │◄───────┤ + Auth/     ├───────►│ (AES-256     │
          │(API Key +   │        │   Rate      │        │  Encryption  │
          │ Rate Limit) │        │   Limiting  │        │  at Rest)    │
          └─────────────┘        │ + Input     │        └──────────────┘
                                 │   Validation│
          ┌─────────────┐        └──────┬──────┘
          │   Phone     │               │
          │Verification │        ┌──────▼──────┐        ┌──────────────┐
          │  Service    │        │ Application │        │ SIEM / Audit │
          └─────────────┘        │   Backend   │───────►│ Logging      │
                                 └──────┬──────┘        │ (CloudWatch/ │
                                        │               │  Splunk)     │
                                 ┌──────▼──────┐        └──────────────┘
                                 │  SQL Queries │
                                 │  Prepared    │
                                 │  Statements  │
                                 │  + ORM       │
                                 └──────┬──────┘
                                        │
                                 ┌──────▼──────┐        ┌──────────────┐
                                 │User Database│        │Admin Dashboard│
                                 │  MySQL DB   │        │ (VPN-Only +  │
                                 │ (Encrypted) │        │  MFA + RBAC) │
                                 └─────────────┘        └──────────────┘
```

---

## Annotated Corrections (Minimum 3 Required)

### Correction 1: HTTP → HTTPS/TLS Encryption for All Data Transmission

**What was changed:** Replaced the unencrypted HTTP connection between the User's Browser/Mobile App and the Load Balancer with **HTTPS using TLS 1.2+** encryption. Added SSL termination at the load balancer and enforced **HTTP Strict Transport Security (HSTS)** headers.

**Why this change was made:** The original architecture transmitted all data — including PII (names, emails, phone numbers, company details) and user credentials — in plaintext over HTTP. This means any attacker positioned on the network (e.g., on a public Wi-Fi at a conference, which is precisely where ConnectSphere's target audience operates) could intercept this data using packet sniffing tools. This directly violates the **CIA Triad's Confidentiality** principle and fails to meet **GDPR Article 32** requirements for encryption of personal data in transit. Implementing HTTPS ensures end-to-end encryption, protecting user PII from eavesdropping and man-in-the-middle attacks.

---

### Correction 2: SQL Queries with No Prepared Statements → Parameterized Queries with ORM

**What was changed:** Replaced raw SQL queries (with no prepared statements) with **parameterized queries using prepared statements and an ORM** (Object-Relational Mapping) layer. All user-supplied input is now sanitized and parameterized before reaching the database.

**Why this change was made:** The original architecture explicitly noted "SQL Queries: No prepared statements," meaning user input was concatenated directly into SQL query strings. This is the textbook definition of a **SQL Injection vulnerability** — one of the OWASP Top 10 most critical web application security risks. An attacker could inject malicious SQL code through any input field (e.g., registration forms, search fields, messaging) to extract, modify, or delete the entire database. This violates the **CIA Triad's Integrity** principle and constitutes a fundamental **Secure SDLC** failure. Parameterized queries ensure that user input is always treated as data, never as executable SQL code, eliminating the injection vector entirely.

---

### Correction 3: Public Admin Dashboard (No Auth) → VPN-Restricted Access with MFA and RBAC

**What was changed:** Moved the Admin Dashboard from a publicly accessible endpoint with no authentication to a **VPN-restricted, MFA-protected endpoint with Role-Based Access Control (RBAC)**. The admin interface is no longer accessible from the public internet.

**Why this change was made:** The original architecture exposed the Admin Dashboard publicly with no authentication whatsoever. This means anyone who discovered the admin URL could access full administrative functions — managing users, modifying events, accessing all PII, or shutting down the platform. This is an extreme violation of the **Principle of Least Privilege (PoLP)**, which dictates that users should only have the minimum access necessary for their role. Unauthenticated public access to administrative controls also violates the **CIA Triad's Availability** principle, as an attacker could disable the platform entirely. The corrected architecture restricts admin access to VPN connections only, requires multi-factor authentication, and enforces role-based permissions so that even authenticated internal users can only perform actions authorized for their specific role.

---

### Correction 4: Added Centralized Audit Logging and SIEM Integration

**What was changed:** Added a **SIEM (Security Information and Event Management) / Audit Logging** component connected to the Application Backend. All authentication events, API access, data modifications, and administrative actions are now logged to a centralized, tamper-resistant logging system (e.g., AWS CloudWatch or Splunk).

**Why this change was made:** The original architecture had no incident response or logging mechanisms in place. Without audit logging, ConnectSphere would have no way to detect active attacks, investigate breaches after they occur, or demonstrate compliance during audits. This is a critical gap for **GDPR compliance** (Article 33 requires breach notification within 72 hours — impossible without detection capabilities) and fundamental **cybersecurity governance**. Centralized logging enables real-time threat detection, forensic investigation, and regulatory compliance reporting.

---

### Correction 5: Public-Facing EC2 Instances → Private Subnet Deployment

**What was changed:** Moved the Web Server EC2 instances from **public-facing** subnets to **private subnets** within the VPC. Only the Load Balancer remains internet-facing, acting as the single point of entry.

**Why this change was made:** In the original architecture, the web servers were deployed on public-facing EC2 instances, meaning they had direct internet-accessible IP addresses. This unnecessarily expands the attack surface — an attacker could bypass the load balancer entirely and target individual server instances. By placing the web servers in private subnets, we reduce the attack surface to a single controlled entry point (the load balancer), applying the **defense-in-depth** strategy and the **Principle of Least Privilege** to network architecture. This is a standard AWS security best practice.
