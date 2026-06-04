# claimflow — Complete Finding Table (34 confirmed)

*Expanded from the RedPick blog post [ProjectDiscovery Vibe-Coding: 152 Findings, 0 FP](https://redpick.ai/blog/redpick-vibeapps-benchmark). Tabular data: [claimflow-findings.csv](claimflow-findings.csv).*

**Application stack**: SvelteKit + Drizzle ORM + SQLite
**Ground-truth**: 24 / 24 (100%)
**Additional code-backed findings**: 10
**Total confirmed**: 34
**Ratio vs answer key**: 1.4×
**False positives**: 0

## 24/24 ground-truth matches

| ID | Sev | Title | Neo? |
| -- | --- | ----- | :--: |
| CF-001 | Critical | Missing Authorization on Admin User Management Actions | ✓ |
| CF-002 | Critical | Deactivated User Retains Full Application Access | ✓ |
| CF-003 | High | IDOR — Document Verification No Ownership Check | ✓ |
| CF-004 | High | Password Hash Exposure via Drizzle ORM Relations | ✓ |
| CF-005 | High | Missing Authorization on Adjuster Claim Actions | ✓ |
| CF-006 | High | Missing Authorization on Underwriter `makeDecision` Action | ✓ |
| CF-007 | Medium | Mass Assignment via Workflow `updateData` Spread | ✓ |
| CF-008 | Medium | Adjuster `updateStatus` Bypasses Workflow Transition Validation | ✓ |
| CF-009 | Medium | Batch Operations Accessible to Adjuster + Bypass Workflow Rules | ✓ |
| CF-010 | Medium | Missing Authorization on Settlement `Calculate` Action | ✓ |
| CF-011 | Medium | Missing Role Check on Triage API | ✓ |
| CF-012 | Medium | Missing Role Check on Settlement/Workflow GET Endpoints | ✓ |
| CF-013 | Medium | CSRF Protection Missing on API Endpoints | ✓ |
| CF-014 | Medium | Unrestricted File Upload in Message Attachments | ✓ |
| CF-015 | Medium | Weak Password Hashing — SHA-256 with Hardcoded Static Salt | ✓ |
| CF-016 | Low | Session Cookie Missing Secure Flag | ✓ |
| CF-017 | Low | Sessions Not Invalidated on Password Change | ✓ |
| CF-018 | Low | No Rate Limiting on Login | ✓ |
| CF-019 | Low | Unlimited Concurrent Sessions | ✓ |
| CF-020 | Low | Account Enumeration via Registration Error Messages | ✓ |
| CF-021 | Info | HSTS Not Enabled | ✓ |
| CF-022 | Info | CSP Not Implemented | ✓ |
| **CF-023** | **Info** | **Generic Email Address Disclosure** | **✗** |
| **CF-024** | **Info** | **X-Content-Type-Options Not Implemented** | **✗** |

## 10 additional code-backed findings

| # | Sev | Title | Source |
| - | --- | ----- | ------ |
| E-CF-01 | **Critical** | IDOR — Cross-User Claim Action Execution via Direct Object Reference | `adjuster/claims/[id]/+page.server.ts:52,119,142,203` |
| E-CF-02 | Medium | Stored CRLF Injection in User Profile Fields | `settings/+page.server.ts`, `register/+page.server.ts` |
| E-CF-03 | Medium | Registration Endpoint Creates Account Despite Returning Error | `register/+page.server.ts:61-64` |
| E-CF-04 | Medium | Weak Password Policy — No Complexity Requirements Beyond Minimum Length | `register/+page.server.ts` |
| E-CF-05 | Medium | Business Logic — Claim Amount Exceeds Policy Coverage Limit | `claims/new/+page.server.ts` |
| E-CF-06 | Low | Sensitive Data Cacheable — Missing Cache-Control on HTML Responses | response headers |
| E-CF-07 | Low | Business Logic — File Upload Content-Type Mismatch Accepted | `/api/documents` |
| E-CF-08 | Low | External Resource Loaded Without Subresource Integrity (SRI) | Google Fonts |
| E-CF-09 | Info | Framework and Technology Disclosure via Headers and `_app/version.json` | `x-sveltekit-page` |
| E-CF-10 | Info | SvelteKit Build Metadata and Environment Endpoint Disclosure | `/_app/version.json`, `/_app/env.js` |

## The most architecturally interesting finding in the benchmark

**CHAIN-001 on claimflow** leans on a SvelteKit-specific behavior: every authenticated route exposes a `__data.json` sibling that returns the server's serialized page-load data.

For `/admin/users`, that serialized data includes password hashes that the HTML page had already sanitized out. The root layout at `+layout.server.ts:5` returns the full `locals.user` into page data, and the session loader at `auth.ts:57` uses `with: { user: true }` to eagerly load the full user relation.

Chained with SHA-256 + static salt (CF-015), offline cracking mass-compromises every account. This attack requires understanding:

1. SvelteKit's `__data.json` mechanism (framework-specific)
2. Drizzle ORM eager-loading with `with: { user: true }`
3. How `locals.user` propagates through layouts
4. Weak password hashing + static salt cracking

This is the type of finding that benchmarks typically miss because it crosses framework boundaries — the vulnerability only materializes when you combine framework behavior with ORM configuration with credential storage.

## Action-layer authorization gaps

The other major category of findings in claimflow is **SvelteKit action-layer authorization gaps**. SvelteKit's `+page.server.ts` files define `actions` that are form-post endpoints. These actions can be called directly (bypassing any UI role check) via their action URL.

E-CF-01 (Critical, IDOR) exploits this: `adjuster/claims/[id]/+page.server.ts` defines `recommendPayout`, `addNote`, `requestDocuments`, and `settlement/calculate` actions that all accept an arbitrary `[id]` without scoping the adjuster to their assigned claims.

The admin page (`admin/users/+page.server.ts`) gates the UI on role, but the form actions do not re-verify it server-side — so a non-admin session can call them directly via the action URL.

## Framework-specific leaks

Several findings exploit SvelteKit-specific behavior:

- **`__data.json` siblings**: expose full serialized page data for any route
- **`/_app/version.json`**: reveals build ID and framework version
- **`/_app/env.js`**: in some configurations, leaks environment variables
- **`x-sveltekit-page` response header**: discloses framework version

These are not bugs in SvelteKit — they're intentional framework behaviors that developers must explicitly handle. But AI-generated SvelteKit code typically does not handle them, leaving the defaults exposed.
