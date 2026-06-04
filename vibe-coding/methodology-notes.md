# Vibe-Coding Benchmark — Methodology Notes

*Expanded from the RedPick blog post [ProjectDiscovery Vibe-Coding: 152 Findings, 0 FP](https://redpick.ai/blog/redpick-vibeapps-benchmark).*

Four architectural decisions separate this run from a conventional single-agent scan. We describe them at the level of *what* they do, not *how* they are implemented — the implementation is proprietary.

## Parallel multi-agent testing with free-roaming coverage

The testing phase fans out into multiple parallel agent groups, each specialized in a different vulnerability class (injection, auth, access control, business logic, client-side, infrastructure, etc.). Each agent runs in its own isolated context so that one agent's reasoning cannot bias another's.

Critically, after the scoped agents complete, a separate **unconstrained agent** reviews everything the structured pass has already found and goes hunting for real vulnerabilities the answer key does not enumerate. This is where most of the 78 additional findings come from. Traditional benchmark agents stop at the ground-truth coverage target; ours does not.

## Strict finding classification

Our scorer splits every finding that does not map to the ground-truth answer key into three categories:

- **Confirmed**: structurally complete, non-duplicate, code-reconciled. Counts toward the total.
- **Near-duplicate**: real bug but same root cause already counted. Does not count toward total.
- **Observation**: stored-but-no-sink, composite-only, or not confirmed by source review. Does not count toward total.

Only confirmed findings count toward the total. This is what lets us report **100% precision honestly**.

### The 41 supporting observations

During testing the pipeline also produced **41 additional items** that we logged as supporting observations but chose **not to count** as confirmed distinct vulnerabilities. Transparency matters more than a larger number:

| Category | Count | Why not counted |
| -------- | :-: | --------------- |
| **Stored unsanitized data — no rendering sink** | 13 | The backend stores unescaped user input (e.g. XSS payloads in transaction descriptions, bill-pay fields, dispute notes). However, the current React and Next.js frontends render these values in normal JSX text nodes — not `dangerouslySetInnerHTML` — so the browser auto-escapes them. They are real hardening gaps but not exploitable in the current codebase. |
| **Near-duplicates / composites** | 23 | Multiple parallel agents independently reported the same root cause under different testing categories. Per our dedup policy ("same endpoint AND same attack chain = duplicate"), these collapse into findings already counted. |
| **Claims not confirmed by source review** | 5 | Source review found no executing sink for the claimed vulnerability. For example: notification `link` field is stored but the current notification UI does not render it as a clickable element; stored URL fields without any server-side fetch. |

The 41 observations are preserved internally with the same raw HTTP evidence as the confirmed findings.

## Sink-pinned code reconciliation

Every additional finding undergoes a source-code reconciliation step before promotion. The pipeline traces from the HTTP-layer PoC back to the specific file and line in the benchmark source where the vulnerable pattern lives.

A finding without a confirmed sink — for example, an XSS claim where the frontend renders the value in a normal React text node rather than `dangerouslySetInnerHTML` — is downgraded to the observation category regardless of whether the PoC "worked" at the HTTP level.

This step is **conservative by design**. The reconciliation uses the benchmark source at commit `17568bbb` as the single source of truth — not the HTTP response, not the AI's reasoning about the response, and not the finding's own title.

## Dual verification

A single verification pass is not enough. On vaultbank, two initially unconfirmed findings were promoted to confirmed only after a second, independent verification attempt.

Our pipeline always runs **two full verification passes**:
- The second is idempotent (already-verified findings stay confirmed)
- Catches edge cases the first pass missed
- Particularly important for race conditions and timing-dependent bugs

## Environment integrity gates

Two classes of setup failures silently destroy a benchmark run:

- **Stale state**: the pipeline resumes from a previous run's data instead of starting clean
- **Broken credentials**: a single role with a dead login starves an entire testing phase

Our pipeline enforces hard pre-flight checks on both — a single failure aborts the run before any testing begins, with a per-role diagnostic.

## Resilient execution

Testing agents never permanently fail on transient errors. Rate-limit responses are parsed and the agent sleeps until the limit resets, then resumes with a full budget.

On medportal, two agents that hit a rate window mid-run silently paused and resumed automatically, completing coverage that a less resilient architecture would have left half-done.

## Attack chain composition

Individual findings are not the point; the point is what they compose into. After verification, a dedicated chain-correlation step looks for sequences of single findings that together form an end-to-end attack.

### 10 chains surfaced

**vaultbank — teller/manager compositional chains** (3 chains): E-VB-01/02/03 + threshold-bypass + race-condition. Collectively they let a teller write themselves unbounded balance and, via transfer splitting, exfiltrate it without hitting any approval gate.

**medportal — 4 chains**:

| ID | Severity | Chain summary |
| -- | :------: | ------------- |
| CHAIN-001 | **Critical** | Patient mass assignment → admin account hijack |
| CHAIN-003 | **Critical** | Patient allergy removal → unsafe prescription approved by medical workflow |
| CHAIN-004 | **High** | CSRF + mass assignment → cross-origin medical record tampering |
| CHAIN-005 | **Critical** | Default credentials + no rate limiting + hash disclosure → full compromise |

CHAIN-001 is the sharpest illustration of the mass-assignment pattern. The patient modifies their own `userId` in `PATCH /api/patients/:id`, then associates their patient profile with the admin's user account. Combined with the lack of ownership checks on every medical endpoint, the patient now reads and writes the admin's medical data, appointments, and prescriptions. **Three individually Medium findings compose into a Critical privilege-boundary breach.**

**claimflow — 3 chains**:

| ID | Severity | Chain summary |
| -- | :------: | ------------- |
| CHAIN-001 | **Critical** | Password hash exposure (`__data.json`) + weak hashing (SHA-256 static salt) + weak policy = mass account takeover |
| CHAIN-002 | **Critical** | Unauthenticated data endpoints + broken access control = full data exfiltration |
| CHAIN-003 | **Critical** | Auth weakness + BFLA + privilege escalation = admin compromise |

See [claimflow-findings.md](claimflow-findings.md) for the architectural details of CHAIN-001.

## The common pattern across all three apps

There is a single recurring shape behind most of the additional findings: **AI-generated code tends to place role checks at the page or UI layer and then pass the request body straight into ORM write calls at the action or API layer.**

Those two patterns together — UI-layer authorization plus full-body spread into `update(...)` — create most of the additional surface we reported.

This is a useful signal for anyone planning a review of similar AI-generated codebases: **the interesting attack surface is almost never the login page; it is the `+page.server.ts` / `route.ts` action handler one level below it.**

## Why this matters for benchmark evaluation

The answer key is a floor, not a ceiling. Each app has a meaningful tail of real vulnerabilities the benchmark does not catalog, and an honest evaluation of a tool must show them too.

Our 152 total confirmed findings vs the 74-item answer key isn't a claim that the answer key is wrong. It's a claim that the answer key is **scoped** — and tools that stop at the answer key (correctly, for benchmark evaluation purposes) miss the real-world attack surface that extends beyond it.

For production engagements, the scope is always the full attack surface, not a curated subset. A tool that performs well on ground-truth coverage but cannot extend beyond it is optimized for the wrong metric.
