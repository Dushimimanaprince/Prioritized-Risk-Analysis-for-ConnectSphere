# Deliverable 3: Summary of Review Process

## ConnectSphere — Set 1: Professional Event Networking MVP

---

## Review Process Summary (200–300 words)

My review of ConnectSphere's architecture began with a systematic assessment guided by core cybersecurity governance principles — the CIA Triad (Confidentiality, Integrity, Availability), the Principle of Least Privilege (PoLP), and Secure SDLC practices. As an independent consultant, I approached the evaluation by mapping each architectural component against these frameworks to identify where the platform's design fell short of industry security standards.

I started by tracing the data flow from the user's browser through the entire system, evaluating each connection point for encryption, authentication, and access control. The most immediate finding was the use of unencrypted HTTP for transmitting PII — a direct Confidentiality violation with serious GDPR implications. Next, I examined the database layer, where the absence of prepared statements exposed the platform to SQL injection attacks, compromising data Integrity. I then assessed the admin dashboard's publicly accessible, unauthenticated endpoint — a severe PoLP violation that jeopardized Availability and overall system security.

These findings directly improve ConnectSphere's compliance and data protection posture. Implementing HTTPS ensures GDPR Article 32 compliance for encryption in transit. Adopting parameterized queries eliminates the most common web application vulnerability (OWASP Top 10 A03:2021 — Injection). Restricting admin access behind VPN, MFA, and RBAC establishes proper access governance.

My recommended monitoring metric — tracking failed authentication and unauthorized access attempts via real-time dashboards — supports ethical and transparent cybersecurity governance by providing continuous visibility into the platform's security state. This enables proactive incident detection, supports GDPR's 72-hour breach notification requirement (Article 33), and demonstrates to investors and auditors that ConnectSphere takes a governance-first approach to security, building the trust necessary for a successful Series A due-diligence review.
