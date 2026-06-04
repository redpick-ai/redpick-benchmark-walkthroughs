# Scoring methodology

How RedPick's runs against Photoview and Fider were scored against the reconstructed Doyensec-derived answer key.

## Two terms (counted on different bases)

- **Internally confirmed** — a *finding* count: deduplicated findings RedPick's agents reproduced with evidence during the run. The total is **62** (28 Photoview + 34 Fider), of which **20** map to reconstructed key items and **42** are extras outside the key (62 = 20 + 42).
- **Found / partial / missed** — *key-item* counts against the 33 reference items. The 21 found + 10 partial items are covered by the 20 key-mapped findings — fewer findings than items, because one finding can satisfy several key items.
- **Extended true positive (ETP)** — the scoring measure that fuses the two: the **21** key items found at full credit plus the **34** extras the scorer validated = **55**. Partial key matches and lower-value or near-duplicate extras do not count toward the 55.

**Every comparison against Doyensec's counts uses the stricter 55.** (Fider's raw dedup produced 37 candidates; 3 low-quality near-duplicates failed the scoring gate and are excluded from the 34 confirmed.)

## The adjusted-score formula

Each reconstructed key item is worth one point:

- full match → **1.0**
- partial match → **0.5**
- miss → **0**

Each **validated extra** (a real finding outside the key that the scorer credits) adds one point to **both** the numerator and the denominator:

```
adjusted score = (key points earned + valid extras) / (key items + valid extras)
```

Because every valid extra enters both sides of the fraction, the score cannot be inflated by piling on noise: an unsubstantiated extra earns nothing, and a weak one the scorer rejects never reaches the numerator. The metric rewards real coverage of the known surface **and** substantiated discovery beyond it, on one normalized scale.

## Per-app computation

- **Photoview:** (12.5 + 18) / (16 + 18) = **30.5 / 34 = 89.7%**
  Key points: 10 found × 1.0 + 5 partial × 0.5 = 12.5.
- **Fider:** (13.5 + 16) / (17 + 16) = **29.5 / 33 = 89.4%**
  Fider's key points use the **scored** 13.5/17, which credits one candidate partial on top of the 13.0 confirmed-only baseline (see note below).
- **Combined:** (26.0 + 34) / (33 + 34) = **60.0 / 67 = 89.6%**
  Combined key points 26.0 = 12.5 (Photoview) + 13.5 (Fider, scored).

89.6% is the share of all available credit earned across 67 points total — the 33 known key items plus the 34 extras RedPick substantiated. The 10.4-point gap from 100% is entirely the 10 partial matches and 2 misses against the key (7.0 lost key points spread across the larger denominator).

### Note on Fider's two key scores

Fider reports both a **strict key score (confirmed-only) of 13.0/17** and a **scored key score of 13.5/17**. The difference is a single candidate partial (server-side template injection): the confirmed-only view counts 11 full matches + 4 confirmed partials = 13.0; the scored view additionally credits the SSTI candidate at 0.5 → 13.5. The combined *strict* score (25.5/33) uses Fider's 13.0; the combined *adjusted* score uses Fider's 13.5. Both are reported so the distinction is explicit.

## Partial / miss ledger

### Misses (2)

| Target | Missed item | Severity | Read |
| --- | --- | :---: | --- |
| Photoview | Missing rate limiting on auth / high-value operations | Low | Related rate-limit primitives found, but not in the exact shape the key item expects. |
| Fider | RCE reachable from an injection primitive | High | No confirmed RCE proof produced from the injection primitive. |

### Fider partials (5)

| Item | Why it was partial |
| --- | --- |
| Pre-auth mass assignment in sign-in binder | Hit the sign-in/authentication surface, but did not prove binder over-posting and sensitive state change strongly enough. |
| Server-side template injection | Candidate/unconfirmed evidence matched SSTI metadata, but the confirmed finding lacked template-evaluation proof. (This is the candidate partial behind the 13.0 → 13.5 difference above.) |
| Open redirect in login / redirect flow | OAuth/redirect probes found adjacent redirect-flow behavior, but not a clean attacker-controlled external redirect proof. |
| Admin-only / privileged workflow exposed | Access-control evidence existed, but not the exact privileged-workflow proof shape the key expects. |
| Race condition in state-changing workflow | Business-logic evidence existed, but no clean concurrent invariant violation was proven. |

### Photoview partials (5)

Directory listing / file enumeration · application technology / version disclosure · broken session management · missing/incorrect cookie security flags · missing security headers / weak TLS configuration. (See [`photoview-findings.md`](./photoview-findings.md) for the per-item key table.)

## Improvements fed back

The partials and misses drove general (not target-specific) changes to RedPick's test runbooks and scoring gates:

- mass-assignment testing must explicitly prove accepted over-posted sensitive fields;
- SSTI testing must separate template evaluation, injection primitive, and RCE escalation;
- redirect testing must require an attacker-controlled external sink, not just redirect-adjacent behavior;
- race testing must prove a broken invariant under concurrency, not only timing or 500-class instability;
- privileged-workflow testing must preserve role and endpoint evidence in a way the final scorer can map cleanly.
