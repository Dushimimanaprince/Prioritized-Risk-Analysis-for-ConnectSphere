# Corrected Architecture — ConnectSphere User Flow (Set 1)

## Description

This file documents the corrected architecture after applying cybersecurity governance recommendations.

## Corrections Applied

| # | Component | Original (Flawed) | Corrected | Principle |
|---|---|---|---|---|
| 1 | Client ↔ LB | HTTP (Unencrypted) | HTTPS / TLS 1.2+ with HSTS | CIA Triad — Confidentiality |
| 2 | Database Layer | No Prepared Statements | Parameterized Queries + ORM | CIA Triad — Integrity, Secure SDLC |
| 3 | Admin Dashboard | Public, No Auth | VPN-Only + MFA + RBAC | PoLP, CIA Triad — Availability |
| 4 | Monitoring | None | SIEM + Centralized Audit Logging | Incident Response, GDPR Art. 33 |
| 5 | Web Servers | Public-Facing EC2 | Private Subnet EC2 | PoLP, Defense in Depth |

## Corrected Architecture Flow

```
User → HTTPS (TLS 1.2+) → Load Balancer (SSL Termination + HSTS) → Private Subnet EC2 Web Servers
    → API Gateway (Auth + Rate Limiting + Input Validation) → Application Backend → MySQL DB (Prepared Statements + ORM, Encrypted at Rest)
    → SIEM / Audit Logging (CloudWatch / Splunk)
    → Admin Dashboard (VPN + MFA + RBAC, Internal Only)
    → SMS Gateway API (API Key + Rate Limit)
```

## Compliance Alignment

- **GDPR Article 32:** Encryption in transit (HTTPS) and at rest (AES-256)
- **GDPR Article 33:** Breach detection via SIEM enables 72-hour notification
- **CCPA:** Proper access controls protect consumer data rights
- **OWASP Top 10:** Injection (A03:2021) mitigated via parameterized queries
