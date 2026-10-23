# Original Architecture — ConnectSphere User Flow (Set 1)

## Description

This file documents the original (flawed) architecture as provided in the assignment brief.

## Identified Flaws

| # | Component | Flaw | Severity |
|---|---|---|---|
| 1 | Client ↔ Load Balancer | HTTP (Unencrypted) — no TLS/SSL | 🔴 Critical |
| 2 | Database Layer | SQL Queries with No Prepared Statements | 🔴 Critical |
| 3 | Admin Dashboard | Public Access, No Authentication | 🔴 Critical |
| 4 | Web Servers | Public-Facing EC2 Instances (exposed IPs) | 🟠 High |
| 5 | SMS Gateway API | No API authentication or rate limiting noted | 🟠 High |
| 6 | Overall System | No audit logging or incident response | 🟠 High |
| 7 | User Database | No encryption at rest indicated | 🟡 Medium |
| 8 | Session Management | No secure session handling noted | 🟡 Medium |

## Architecture Flow

```
User → HTTP (plain) → Load Balancer → Public EC2 Web Servers → API Gateway → Application Backend → MySQL DB (No prepared statements)
                                                                                                  ↘ Admin Dashboard (Public, No Auth)
                       SMS Gateway API (No Auth) ← API Gateway
                       Phone Verification Service ← Backend
```
