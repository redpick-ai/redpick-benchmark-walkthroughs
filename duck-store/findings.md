# Duck Store — full findings

Source-free grey-box run against Escape's hosted Duck Store (`https://duck-store.escape.tech/`) under article-parity inputs: URL + `/openapi.json` pointer + default credentials (`admin/admin`, `user/password`), no source code, no answer-key access during testing.

Reference: Escape, ["Benchmarking AI Pentesting Tools: A Practical Comparison"](https://escape.tech/blog/benchmarking-agentic-ai-pentesting-tools/) (2026-04-30). RedPick was not part of Escape's study; the counts here are RedPick's own scorer output.

**Strict article-key score: 20 / 20 (100.0%)** — 0 partial, 0 missed, single ~7h run. Severity mix: 7 Critical, 9 High, 3 Medium, 1 Low.

## The 20 article-parity vulnerabilities

All confirmed with a working, reproducible proof of concept.

| Vulnerability | Severity | CVSS | OWASP | CWE | Endpoint | Result |
| --- | :---: | :---: | :---: | :---: | --- | :---: |
| Weak admin credentials | Critical | 9.8 | A07 | CWE-521 | `/api/v1/auth/login` | Found |
| JWT `alg=none` accepted | Critical | 9.8 | A07 | CWE-347 | Any authenticated endpoint | Found |
| 2FA / TOTP bypass | Critical | 9.1 | A07 | CWE-287 | `/api/v1/auth/login/totp` | Found |
| IDOR on user profiles | Critical | 9.1 | A01 | CWE-639 | `/api/v1/users/{uuid}` | Found |
| Mass assignment of profile role | Critical | 9.1 | A01 | CWE-915 | `/api/v1/users/me/profile` | Found |
| 100% discount coupon abuse | Critical | 9.1 | A04 | CWE-840 | `/api/v1/orders/checkout` | Found |
| Referral code abuse | Critical | 9.1 | A04 | CWE-840 | `/api/v1/auth/register` | Found |
| SQL injection in product color filter | High | 8.1 | A03 | CWE-89 | `/api/v1/products/filter/by-color` | Found |
| Stored XSS in testimonials | High | 8.1 | A03 | CWE-79 | `/testimonials` | Found |
| SSRF via image import | High | 8.1 | A10 | CWE-918 | `/api/v1/uploads/import-from-url` | Found |
| SSRF via link-preview fetch | High | 8.1 | A10 | CWE-918 | `/api/v1/uploads/fetch-url` | Found |
| Unauthenticated testimonial modification | High | 8.1 | A01 | CWE-862 | `/api/v1/testimonials/{id}` | Found |
| IDOR on order details | High | 8.1 | A01 | CWE-639 | `/api/v1/orders/{id}` | Found |
| Broken access control on admin users | High | 8.1 | A01 | CWE-862 | `/api/v1/admin/users` | Found |
| Negative-quantity cart logic | High | 8.1 | A04 | CWE-840 | `/api/v1/cart/add` | Found |
| Shipping-cost bypass | High | 8.1 | A04 | CWE-840 | `/api/v1/orders/checkout` | Found |
| Coupon information disclosure | Medium | 6.5 | INFO | CWE-200 | `/api/v1/orders/coupons` | Found |
| No rate limiting on login | Medium | 6.5 | A07 | CWE-307 | `/api/v1/auth/login` | Found |
| Open redirect | Medium | 6.1 | A05 | CWE-601 | `/...?redirect=` | Found |
| User enumeration | Low | 3.7 | INFO | CWE-203 | `/api/v1/users/` | Found |

By category: A01 Broken Access Control 5 · A03 Injection 2 · A04 Insecure Design/Business Logic 4 · A05 Misconfiguration/Open Redirect 1 · A07 Identification & Authentication 4 · A10 SSRF 2 · INFO Information Disclosure 2. By difficulty: Hard 2 · Medium 8 · Easy 10.

## Validated extras beyond the key (deduplicated)

The scorer credited **88 extra finding files** beyond the 20-item key. Those are finding *files*, not 88 distinct bugs: multiple specialist agents and both engines re-reported the same issues, so they collapse to **26 distinct vulnerability classes**. About half (~49 files) are additional instances or deeper exploitation of the 20 scored items; **12 classes are genuinely new** beyond the article's key (marked **New**).

| Vulnerability class | Severity | Finding files | Beyond the 20-item key | Example endpoint |
| --- | :---: | :---: | :---: | --- |
| Mass assignment → privilege escalation (profile role) | Critical | 8 | Additional | `PUT /api/v1/users/me/profile` |
| JWT `alg=none` / algorithm confusion | Critical | 7 | Additional | every JWT-protected route |
| Shadow product API — non-admin create / modify | Critical | 5 | **New** | `POST`/`PUT` `/api/v1/products/` |
| SSRF (link-preview fetch) + cloud-metadata chain | Critical | 5 | Additional | `GET /api/v1/uploads/fetch-url` |
| SQL error / verbose info disclosure | Critical | 2 | Additional | `GET /api/v1/products/filter/by-color` |
| BOLA / IDOR (orders, testimonials, users) | High | 9 | Additional | `/api/v1/orders/{id}` |
| Race conditions (checkout / orders / reviews) | High | 5 | **New** | `POST /api/v1/orders/checkout` |
| BFLA on admin user listing / management | High | 4 | Additional | `GET /api/v1/admin/users` |
| Broken access control (testimonials) | High | 4 | Additional | `/api/v1/testimonials/{id}` |
| Private coupon disclosure | High | 2 | Additional | `GET /api/v1/orders/coupons` |
| Stored / DOM XSS (additional vector) | High | 1 | Additional | `/admin/users` |
| Cart input validation / business logic | High | 1 | Additional | `POST /api/v1/cart/add` |
| SQL injection (additional vector) | High | 1 | Additional | `/api/v1/products/filter/by-color` |
| Client-controlled shipping cost | High | 1 | Additional | `POST /api/v1/orders/checkout` |
| Mass assignment of `is_featured` (testimonials) | High | 1 | **New** | `POST /api/v1/testimonials/` |
| No password policy / weak passwords accepted | High | 1 | Additional | `POST /api/v1/auth/register` |
| DOM-based open redirect (SPA routes) | Medium | 15 | **New** | `/cart/{id}`, `/reviews/{id}`, `/admin/{id}` |
| User / PII enumeration (users API) | Medium | 3 | Additional | `GET /api/v1/users/` |
| Missing security / hardening headers | Medium | 3 | **New** | site-wide |
| No logout / no JWT revocation | Medium | 2 | **New** | auth endpoints |
| Missing Subresource Integrity (SRI) | Medium | 2 | **New** | `GET /` |
| No audit logging | Medium | 1 | **New** | application-wide |
| Cleartext credentials via HTTPS→HTTP redirect | Medium | 1 | **New** | `/api/v1/auth/login` |
| SQL wildcard injection (product search) | Medium | 1 | **New** | `GET /api/v1/products/search/` |
| Upload content-validation bypass | Low | 2 | **New** | `/api/v1/uploads/avatar` |
| Verbose errors / stack-trace disclosure | Low | 1 | **New** | `/api/v1/admin/orders` |

The single largest cluster — 15 DOM-based open-redirect files across different SPA routes — is one underlying client-side redirect issue re-reported per route. The honest read is "26 distinct extra classes, 12 of them new," not "88 new bugs." See [`scoring-methodology.md`](./scoring-methodology.md) for how the extended scorer credits these.
