# Photoview — full findings

Pure black-box run (URL + credentials only). **28 internally confirmed findings**, splitting **8 / 20** between key-mapped findings and extras. Against the 16-item reconstructed answer key, coverage was **10 ✓ / 5 ⚠ / 1 ✗** (the 8 key-mapped findings cover the 15 found+partial items, since one finding can satisfy several key items), plus **18 validated extras** credited outside the key. Adjusted score **30.5 / 34 (89.7%)**. Throughout, *validated* means credited by RedPick's own scorer on internal evidence — **not** reviewed by Doyensec.

Status legend: **✓ Found** (full credit) · **⚠ Partial** (half credit) · **✗ Missed**.

## Answer-key coverage — 16 items (10 ✓ / 5 ⚠ / 1 ✗)

| # | Reference vulnerability | Category | Severity | Status |
| :---: | --- | --- | :---: | :---: |
| 1 | SQL injection in GraphQL/API data access | Injection | Critical | ✓ |
| 2 | SQL injection chain to non-owned data | Injection | High | ✓ |
| 3 | SSRF in media / URL-fetching | SSRF | Medium | ✓ |
| 4 | IDOR across album / photo boundaries | Access control | High | ✓ |
| 5 | Any user accesses leaked info for non-owned media | Access control | High | ✓ |
| 6 | Broken authorization in private media API | Access control | High | ✓ |
| 7 | GraphQL introspection / configuration exposure | GraphQL | Low | ✓ |
| 8 | GraphQL leaks low-sensitivity object metadata | GraphQL | Informational | ✓ |
| 9 | Verbose API error discloses absolute server path | Information exposure | Informational | ✓ |
| 10 | Directory listing / file enumeration | Information exposure | Low | ⚠ |
| 11 | Minimal sensitive-data leakage via metadata | Information exposure | Informational | ✓ |
| 12 | Application technology / version disclosure | Information exposure | Informational | ⚠ |
| 13 | Broken session management | Auth / session | Medium | ⚠ |
| 14 | Missing rate limiting on auth / high-value operations | Auth / session | Low | ✗ |
| 15 | Missing / incorrect cookie security flags | Misconfiguration | Low | ⚠ |
| 16 | Missing security headers / weak TLS configuration | Misconfiguration | Low | ⚠ |

The single strict miss (#14) is a rate-limiting item. RedPick found related rate-limit primitives in a different shape — the GraphQL alias amplification and missing rate limiting on the authorization mutation (see extras below) — but they did not map strongly enough to the key item.

## Validated extras — outside the key

RedPick confirmed 20 findings outside the answer key; the adjusted score credits the 18 the scorer rated materially valid (the remainder are lower-severity header/cookie hardening or near-duplicates).

| Validated extra | Type | Severity |
| --- | --- | :---: |
| GraphQL aliases amplify `authorizeUser` brute force (≈20× attempts/request) | Access control / brute force | High |
| Empty password accepted on password change and login | Weak password policy | High |
| User mints a public share for an album they don't own (incl. admin-owned) | Broken access control | High |
| No rate limiting on the `authorizeUser` authentication mutation | Missing rate limiting | High |
| GraphQL accepts `multipart/form-data` — CSRF where cookie-based auth is used | CSRF (cookie-auth dependent) | Medium |
| `LIKE` / wildcard filter injection in GraphQL search | Search-filter injection (not classic SQLi) | Medium |
| Public share leaks server file paths and owner metadata | Information exposure | Medium |
| Username enumeration via login timing oracle | User enumeration | Medium |
| Auth token stored in non-`HttpOnly`, XSS-stealable cookie | Session management | Medium |
| No logout / session-invalidation primitive | Session management | Medium |
| Missing media paths trigger a reverse-proxy 502 (deployment / proxy behavior) | Error handling | Medium |
| Missing HSTS header (deployment / edge config) | Transport security | Medium |
| Weak CBC-mode TLS 1.2 cipher suites (BEAST / LUCKY13 family; deployment / edge config) | TLS configuration | Low |
| Missing `frame-ancestors` / `X-Frame-Options` (deployment / edge config) | Clickjacking | Low |
| _(plus cookie-flag and security-header hardening items)_ | Misconfiguration | Low |
