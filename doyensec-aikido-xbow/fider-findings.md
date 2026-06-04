# Fider — full findings

Pure black-box run; live browser-authenticated Google sessions for the admin and member accounts, no repository or source code. Deduplication produced 37 candidate findings; 3 low-quality near-duplicates did not pass the scoring gate, leaving **34 internally confirmed findings**, splitting **12 / 22** between key-mapped findings and extras. Against the 17-item reconstructed answer key, coverage was **11 ✓ / 5 ⚠ / 1 ✗** (the 12 key-mapped findings cover the 16 found+partial items), plus **16 validated extras** credited outside the key. Adjusted score **29.5 / 33 (89.4%)**.

Status legend: **✓ Found** (full credit) · **⚠ Partial** (half credit) · **✗ Missed**.

## Answer-key coverage — 17 items (11 ✓ / 5 ⚠ / 1 ✗)

| # | Reference vulnerability | Category | Severity | Status |
| :---: | --- | --- | :---: | :---: |
| 1 | Pre-auth mass assignment in sign-in binder (account takeover) | Auth / session | Critical | ⚠ |
| 2 | Missing rate limiting on email-link / authentication flows | Auth / session | Medium | ✓ |
| 3 | Cross-site scripting in user-controlled feedback content | Injection | Medium | ✓ |
| 4 | Server-side template injection in rendered content | Injection | High | ⚠ |
| 5 | Remote code execution reachable from injection primitive | Injection | High | ✗ |
| 6 | Path traversal in file / static-resource handling | File / path | Medium | ✓ |
| 7 | SSRF through URL-consuming functionality | SSRF | Medium | ✓ |
| 8 | Open redirect in login / redirect flow | Open redirect | Low | ⚠ |
| 9 | Information disclosure via low-impact oracle | Information exposure | Low | ✓ |
| 10 | Missing / misconfigured security headers | Misconfiguration | Low | ✓ |
| 11 | Authorization bypass between roles | Access control | High | ✓ |
| 12 | Admin-only / privileged workflow exposed | Access control | Medium | ⚠ |
| 13 | Authentication state / session-management weakness | Auth / session | Medium | ✓ |
| 14 | User content sanitization gap | Injection | Medium | ✓ |
| 15 | Business-logic validation error | Insecure design / logic | Medium | ✓ |
| 16 | Race condition in state-changing workflow | Insecure design / logic | Medium | ⚠ |
| 17 | Email-authentication flow weakness (excl. host-header FP) | Auth / session | High | ✓ |

## Answer-key ledger by category

| Category | Total | Found | Partial | Missed |
| --- | :---: | :---: | :---: | :---: |
| Authentication / session | 4 | 3 | 1 | 0 |
| Injection | 4 | 2 | 1 | 1 |
| File/path handling | 1 | 1 | 0 | 0 |
| SSRF | 1 | 1 | 0 | 0 |
| Open redirect | 1 | 0 | 1 | 0 |
| Information exposure | 1 | 1 | 0 | 0 |
| Misconfiguration | 1 | 1 | 0 | 0 |
| Access control | 2 | 1 | 1 | 0 |
| Insecure design / logic | 2 | 1 | 1 | 0 |
| **Total** | **17** | **11** | **5** | **1** |

## Validated extras — outside the key

RedPick confirmed 22 findings outside the answer key; the adjusted score credits the 16 the scorer rated materially valid (the remainder are lower-severity hardening or near-duplicate observations).

| Validated extra | Type | Severity |
| --- | --- | :---: |
| Excessive session / JWT lifetime (`auth` cookie valid 365 days) | Session management | Medium |
| Static avatar / CSS endpoints emit `Set-Cookie` with `Cache-Control: public` (shared-cache session fixation) | Session management | Medium |
| Auth cookies served without a `SameSite` attribute | Session management | Low |
| Production source maps expose client-side source | Information exposure | Medium |
| Admin HTML containing user emails returned without cache-control | Information exposure | Medium |
| `marked` 4.3.0 — known ReDoS in inline tokenizer | Vulnerable component | Medium |
| Missing Subresource Integrity on first-party bundles | Supply chain | Low |
| Missing HSTS header (HTTPS downgrade window) | Transport security | Medium |
| Sign-in API discloses whether an email belongs to an existing user | User enumeration | Low |
| `/api/v1/posts` returns HTTP 500 on negative `limit` (unhandled DB exception, unauth) | Error handling | Low |
| `preventIndexing` does not gate `sitemap.xml` / `robots.txt` (post-URL inventory enumerable) | Information exposure | Low |
| Missing rate limiting on content creation | Missing rate limiting | Low |
| Missing anti-framing controls (clickjacking) | Clickjacking | Low |
| _(plus additional header / cookie hardening items)_ | Misconfiguration | Low |
