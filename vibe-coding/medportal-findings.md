# medportal — Complete Finding Table (59 confirmed)

*Expanded from the RedPick blog post [ProjectDiscovery Vibe-Coding: 152 Findings, 0 FP](https://redpick.ai/blog/redpick-vibeapps-benchmark). Tabular data: [medportal-findings.csv](medportal-findings.csv).*

**Application stack**: Next.js 14.1.4 + Prisma + NextAuth + PostgreSQL
**Ground-truth**: 20 / 20 (100%)
**Additional code-backed findings**: 39
**Total confirmed**: 59
**Ratio vs answer key**: 3.0× (the largest expansion of the three apps)
**False positives**: 0

## 20/20 ground-truth matches

| ID | Sev | Title | Neo? |
| -- | --- | ----- | :--: |
| MED-001 | High | Password Hash Exposure Across Multiple Endpoints | ✓ |
| MED-002 | High | Privilege Escalation via Mass Assignment on User Update | ✓ |
| MED-003 | High | Mass Assignment Across All API Endpoints | ✓ |
| MED-004 | High | IDOR — No Ownership Verification on Patient Data | ✓ |
| MED-005 | High | Middleware Only Protects Dashboard Routes | ✓ |
| MED-006 | Medium | Search API Exposes All Data Without Role Restriction | ✓ |
| MED-007 | Low | No Rate Limiting on Authentication | ✓ |
| MED-008 | Low | Missing Security Headers | ✓ |
| MED-009 | Low | No Session Invalidation | ✓ |
| MED-010 | Low | Weak Default Password in User Creation API | ✓ |
| MED-011 | Low | No Input Length Validation | ✓ |
| MED-012 | Low | Negative and Extreme Numeric Values Accepted | ✓ |
| MED-013 | Low | Empty Required Fields Accepted | ✓ |
| MED-014 | Info | Server Version Disclosure | ✓ |
| MED-015 | Info | HSTS Not Enabled | ✓ |
| MED-016 | Info | CSP Not Implemented | ✓ |
| **MED-017** | **Info** | **Outdated JavaScript Libraries** | **✗** |
| **MED-018** | **Info** | **X-Content-Type-Options Not Implemented** | **✗** |
| **MED-019** | **Info** | **Verbose Error Messages Leak Internal State** | **✗** |
| MED-033 | High | Privilege Escalation — Nurse Creates Prescriptions | ✓ |

## 39 additional code-backed findings

| # | Sev | Title | Source |
| - | --- | ----- | ------ |
| E-MED-01 | **Critical** | Critical Mass Assignment — Patient Can Overwrite `userId` to Hijack Account | `patients/[id]/route.ts:29` |
| E-MED-02 | **Critical** | Broken Access Control — Nurse Can Create Prescriptions (Medical Safety Boundary) | `prescriptions/route.ts` |
| E-MED-03 | **Critical** | Attack Chain Primitive — Patient Self-Modification Enables Allergy Removal | `patients/[id]/route.ts:29` |
| E-MED-04 | High | Sensitive Data Exposure — Password Hash Disclosure via `GET /api/users` | `users/route.ts:13` |
| E-MED-05 | High | Broken Access Control — Unauthorized Role Access to Medical Records | `medical-records/route.ts` |
| E-MED-06 | High | Mass Assignment — Medical Record Arbitrary Field Modification | `medical-records/[id]/route.ts` |
| E-MED-07 | High | Mass Assignment — Prescription Arbitrary Field Modification | `prescriptions/[id]/route.ts` |
| E-MED-08 | High | IDOR — Missing Ownership Check on `DELETE /api/appointments/:id` | `appointments/[id]/route.ts` |
| E-MED-09 | High | Mass Assignment — `POST /api/vitals` Arbitrary Field Injection | `vitals/route.ts:30` |
| E-MED-10 | High | Mass Assignment — `POST /api/patients` Arbitrary Field Injection | `patients/route.ts:28` |
| E-MED-11 | High | Mass Assignment — Appointment Status and Fields Modifiable by Patient | `appointments/[id]/route.ts` |
| E-MED-12 | High | IDOR — Any Patient Can Modify Any Appointment | `appointments/[id]/route.ts` |
| E-MED-13 | High | Mass Assignment — Appointment Creation with Arbitrary Status | `appointments/route.ts` |
| E-MED-14 | High | Privilege Escalation — Lab Tech Can Create Arbitrary Notifications | `notifications/route.ts` |
| E-MED-15 | High | Insecure Default Credentials in User Creation | `users/route.ts` |
| E-MED-16 | High | Broken Access Control — Patient Can Self-Modify Patient Record | `patients/[id]/route.ts` |
| E-MED-17 | High | Business Logic — Share Link Revocation Bypass | `share-links/[id]/route.ts` |
| E-MED-18 | High | Business Logic — Lab Order Workflow State Bypass | `lab-orders/[id]/route.ts` |
| E-MED-19 | High | Business Logic — Appointment Status and Date Manipulation | `appointments/route.ts` |
| E-MED-20 | High | Mass Assignment — Medical Records (alternate path) | `medical-records/[id]/route.ts` |
| E-MED-21 | Medium | IDOR — Missing Ownership Check on `PATCH /api/share-links/:id` | `share-links/[id]/route.ts` |
| E-MED-22 | Medium | Clickjacking — Missing Anti-Framing Controls (All Pages) | response headers |
| E-MED-23 | Medium | No Server-Side CSRF Protection — Content-Type Confusion | request handling |
| E-MED-24 | Medium | Broken Function Level Authorization — Doctor Accessing Admin Endpoint | `change-requests/route.ts` |
| E-MED-25 | Medium | Missing Login Rate Limiting | auth handler |
| E-MED-26 | Medium | Broken Access Control — Nurse Role Exceeds Authorized Scope | multiple routes |
| E-MED-27 | Medium | Cross-Site Request Forgery on State-Changing Endpoints | multiple routes |
| E-MED-28 | Medium | Data Integrity — Vitals Records Without Nurse/Doctor Attribution | `vitals/route.ts` |
| E-MED-29 | Medium | Missing Cache-Control on Sensitive API Responses | response headers |
| E-MED-30 | Medium | Middleware Authentication Gap — API Routes Not Covered by NextAuth | `middleware.ts:4` |
| E-MED-31 | Medium | Business Logic — Appointment Check-In Status Mass Assignment | `appointments/route.ts` |
| E-MED-32 | Medium | Business Logic — Lab Upload Technician ID Override | `lab-results/upload/route.ts` |
| E-MED-33 | Medium | Vulnerable and Outdated Component — Next.js 14.1.4 (CVE-2024-46982) | `package.json` |
| E-MED-34 | Low | Session Cookie Missing Secure Flag | auth cookie |
| E-MED-35 | Low | Unlimited Concurrent Sessions | session management |
| E-MED-36 | Low | Next.js Build ID and Application Structure Disclosure | `/_next/static/` |
| E-MED-37 | Low | Business Logic — Doctor Self-Referral Allowed | `referrals/route.ts` |
| E-MED-38 | Info | User Enumeration via Login | auth handler |
| E-MED-39 | Info | Technology Stack Disclosure via `X-Powered-By` Header | response headers |

## The structural pattern: ORM-level mass assignment

The medportal expansion (39 additional findings — 2.0× the ground-truth set) reflects a **structural mass-assignment pattern** that affects nearly every `POST`/`PATCH` endpoint.

### The root cause

Nearly every Next.js API route passes the request body directly into Prisma's spread operator:

```typescript
// Vulnerable pattern repeated across almost every route handler
await prisma.model.update({
  where: { id },
  data: body  // <-- entire request body
});
```

This means **every writable field in the Prisma schema is writable by every authenticated role** — regardless of whether the role should have access to that field or not.

### Why the answer key tracks only some of these

The answer key (MED-003 "Mass Assignment Across All API Endpoints") correctly captures the *pattern*. But our approach writes a finding for **every independently exploitable instance** because each is:

- Reachable by a different role
- On a different endpoint
- Against a different Prisma model
- With different downstream impact

A user attacked through `POST /api/vitals` is not protected by the fix for `POST /api/patients`. Each instance is separately exploitable and requires separate remediation.

### The most impactful finding

**E-MED-01 (Patient can overwrite `userId`, Critical)** at `patients/[id]/route.ts:29` is the primitive behind CHAIN-001 (mass account takeover). A patient modifies their own `userId` in `PATCH /api/patients/:id`, associating their patient profile with the admin's user account. Combined with the lack of ownership checks on every medical endpoint, the patient now reads and writes the admin's medical data, appointments, and prescriptions.

Three individually Medium findings compose into a Critical privilege-boundary breach — see the chain composition in [methodology-notes.md](methodology-notes.md).
