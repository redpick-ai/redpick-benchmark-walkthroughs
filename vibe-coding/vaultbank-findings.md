# vaultbank — Complete Finding Table (59 confirmed)

*Expanded from the RedPick blog post [ProjectDiscovery Vibe-Coding: 152 Findings, 0 FP](https://redpick.ai/blog/redpick-vibeapps-benchmark). Tabular data: [vaultbank-findings.csv](vaultbank-findings.csv).*

**Application stack**: FastAPI + PostgreSQL + React
**Ground-truth**: 30 / 30 (100%)
**Additional code-backed findings**: 29
**Total confirmed**: 59
**Ratio vs answer key**: 2.0×
**False positives**: 0

## 30/30 ground-truth matches

| ID | Sev | Title | Neo? |
| -- | --- | ----- | :--: |
| VB-001 | Critical | Business Logic — Arbitrary Self-Deposit | ✓ |
| VB-002 | Critical | Hardcoded JWT Secret Key | ✓ |
| VB-003 | Critical | Business Logic — Dispute Reversal Double-Spend | ✓ |
| VB-004 | Critical | Business Logic — Arbitrary Refund Amount in Dispute Resolution | ✓ |
| VB-005 | High | Broken Access Control — Teller Bypasses Transaction Approval Limits | ✓ |
| VB-007 | High | IDOR — Manager Freezes Any Account Without Branch Restriction | ✓ |
| VB-008 | Medium | Unrestricted File Upload in Dispute Evidence | ✓ |
| VB-009 | Medium | WebSocket Token Exposed in URL | ✓ |
| VB-010 | Medium | IDOR — Unvalidated `disburse_to_account_id` | ✓ |
| VB-011 | Medium | Business Logic — Unlimited Loan Amount | ✓ |
| VB-012 | Low | No Password Complexity Requirements | ✓ |
| VB-013 | Low | JWT Stored in localStorage | ✓ |
| VB-014 | Low | Rate Limiting Only on Auth Endpoints | ✓ |
| VB-015 | Medium | Client-Side Only Role Guards | ✓ |
| **VB-016** | **Low** | **Refresh Token Endpoint Has No Rate Limiting** | **✗** |
| **VB-017** | **Low** | **No Account Lockout After Failed Login Attempts** | **✗** |
| **VB-018** | **Low** | **No Audit Logging for Privileged Operations** | **✗** |
| VB-019 | Low | Server Version Disclosure | ✓ |
| VB-020 | Low | Password Change Endpoint Non-Functional | ✓ |
| VB-021 | Low | Missing Security Headers | ✓ |
| VB-022 | Low | Truncated Transaction Reference Numbers | ✓ |
| VB-023 | Low | SQL Wildcard Injection | ✓ |
| VB-024 | Low | Insecure Account Number PRNG | ✓ |
| VB-025 | Low | Host Header Injection Accepted | ✓ |
| VB-026 | Info | HSTS Not Enabled | ✓ |
| VB-027 | Info | CSP Not Implemented | ✓ |
| VB-034 | Critical | Race Condition in Transfer Operations | ✓ |
| VB-039 | Medium | JWT Token Reuse After Logout | ✓ |
| VB-046 | Critical | Broken Access Control — Teller/Manager Operations Lack All Security Controls | ✓ |
| VB-047 | High | IDOR — Cross-User Dispute Filing | ✓ |

## 29 additional code-backed findings

| # | Sev | Title | Source |
| - | --- | ----- | ------ |
| E-VB-01 | **Critical** | Teller Can Deposit and Withdraw to Own Accounts — No Self-Transaction Check | `teller.py:113,157` |
| E-VB-02 | High | Branch Manager Transfers Bypass Approval and Dual-Approval Thresholds | `transactions.py` |
| E-VB-03 | High | Bill Payment Bypasses Transaction Approval Threshold | `transactions.py` |
| E-VB-04 | High | Loan Approval Workflow Bypass — Compliance Review Skipped | `loan_applications.py` |
| E-VB-05 | High | Race Condition in Teller Deposit Creates Phantom Transactions | `teller.py` |
| E-VB-06 | High | Cleartext Transmission of Sensitive Information (No TLS/HTTPS) | deployment |
| E-VB-07 | High | Missing Multi-Factor Authentication (all roles) | `auth.py` |
| E-VB-08 | Medium | Transaction Approval Threshold Bypass via Transaction Splitting | `transactions.py` |
| E-VB-09 | Medium | Unicode Homoglyph Email Registration (Confusable Characters) | `auth.py` |
| E-VB-10 | Medium | Host Header Injection in HTTP 307 Trailing-Slash Redirects (28/29 endpoints) | FastAPI router |
| E-VB-11 | Medium | Improper Authentication — Inactive Account Access | `auth.py` |
| E-VB-12 | Medium | Insufficient Session Expiration | `auth.py` |
| E-VB-13 | Medium | Insufficient Session Management (no concurrent-session controls) | `auth.py` |
| E-VB-14 | Medium | Clickjacking — Missing Frame Protection | response headers |
| E-VB-15 | Medium | WebSocket Cross-Site Hijacking (CSWSH) — Missing Origin Validation | `notifications_ws.py` |
| E-VB-16 | Medium | Missing Cache-Control Headers on Sensitive API Responses | response headers |
| E-VB-17 | Low | Currency Quote Endpoint Accepts Negative Amounts | `currency.py` |
| E-VB-18 | Low | WebSocket Authentication Token in URL Query String | `notifications_ws.py:14` |
| E-VB-19 | Low | CORS Misconfiguration — Permissive Preflight with Credentials | `main.py` CORS config |
| E-VB-20 | Low | Sensitive Data Exposure at Login Response | `auth.py` |
| E-VB-21 | Low | Verbose Error Messages Disclose Framework and Version Details | error handler |
| E-VB-22 | Low | Missing Cache-Control Headers (secondary path) | response headers |
| E-VB-23 | Low | Missing Subresource Integrity (SRI) on External CDN Resource | `index.html` |
| E-VB-24 | Low | Path Traversal in Evidence Upload Filename (Stored) | `disputes.py` |
| E-VB-25 | Low | CRLF Injection in Transaction Description (Stored) | `transactions.py` |
| E-VB-26 | Low | Negative Amount Accepted in Currency Quote (alternate path) | `currency.py` |
| E-VB-27 | Info | Server Version Disclosure (all endpoints) | `Server` header |
| E-VB-28 | Info | Rate Limit Configuration Disclosure | `429` response body |
| E-VB-29 | Info | Web Server Version Disclosure (Server header) | response headers |

## The common pattern

Most findings above require reasoning about **multi-role authorization boundaries** at the action layer rather than at the page layer.

The key architectural issue: the application differentiates between customer role (goes through the approval gate) and teller/manager roles (take a different code path that writes the transaction directly). Role-aware code paths exist but authorization checks do not consistently scope "action X applied to resource Y" to the requesting user.

The most impactful finding — **E-VB-01 (Teller self-transaction, Critical)** — unlocks most of the teller-role attack chains documented in [CHAIN composition](methodology-notes.md).
