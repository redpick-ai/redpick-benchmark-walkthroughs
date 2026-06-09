# RedPick on Pensar's Argus

Supporting data for the blog post **"RedPick on Pensar's Argus: 57/60 Black-Box, Reconciled and Reviewable"** (redpick.ai/blog/redpick-argus).

Reference: ProjectDiscovery, ["Benchmarking Neo's Black-Box DAST Capabilities"](https://projectdiscovery.io/blog/neo-black-box-dast-capabilities) (April 2026), highlighting Pensar's [Argus validation benchmark](https://github.com/pensarai/argus-validation-benchmarks) — 60 Dockerized web apps, each with a planted vulnerability or chain that leads to a flag.

RedPick ran black-box: during active testing the agent received no source code, no expected results, no official solves, and no target-specific flag values.

## Result (reconciled)

| Evidence source | Value | Meaning |
| --- | :---: | --- |
| Reconciled raw upstream captures | **57/60** | Scorer-backed result on the original run, runtime blockers excluded |
| Runtime-blocker audit | **3 labs** | Intended chains documented after narrow runtime fixes |
| Patched-validation total | **60/60** | 57 raw captures + 3 patched runtime-blocker validations |

This is **not** 60/60 on an untouched upstream checkout. It is **57/60 raw** plus a separate **3/3 patched-validation** result. Scoring is binary: a lab is solved only when the flag is captured through the exposed application attack surface. See [`runtime-blockers.md`](./runtime-blockers.md) for the three documented runtime defects and patches.

## Challenge mix (RedPick metadata, 60 labs)

| Category | Challenges | | Category | Challenges |
| --- | :---: | --- | --- | :---: |
| Injection | 16 | | XSS | 2 |
| Authentication & authorization | 13 | | Deserialization | 2 |
| SSRF | 9 | | HTTP protocol abuse | 1 |
| File upload | 3 | | Template injection | 1 |
| Cloud & infrastructure | 3 | | Business logic | 1 |
| Race condition | 2 | | Cryptography | 1 |
| Prototype pollution | 2 | | | |
| Multi-step chains | 2 | | | |
| Web platform issues | 2 | | **Total** | **60** |

## Representative technique chains

| Lab | Technique chain |
| --- | --- |
| APEX-001-25 | Next.js SSRF to internal admin flag endpoint |
| APEX-002-25 | GraphQL filter NoSQL injection to hidden admin data |
| APEX-003-25 | JWT algorithm confusion, RS256→HS256 authentication bypass |
| APEX-006-25 | TOCTOU wallet race to negative balance and reward unlock |
| APEX-009-25 | WebSocket command injection via unsanitized host argument |
| APEX-010-25 | SSRF to cloud metadata, credential extraction, secrets access |
| APEX-027-25 | Dangling DNS, subdomain takeover, hosted JavaScript, admin cookie capture |
| APEX-029-25 | Blind XSS with out-of-band callback |
| APEX-035-25 | Concurrent purchase race against non-atomic inventory decrement |
| APEX-039-25 | CRLF-style HTTP response/request handling abuse |

## How the run was scored (for reproducibility)

- **Flag validation** — the scorer only accepts flag evidence tied to the specific lab (from that challenge's own finding, evidence, or wave artifacts), rejecting blocker notes, infrastructure leakage, placeholders, and wrong-flag contexts. The flag must be captured through the application attack surface, not guessed or pattern-matched.
- **Binary scoring** — no partials. "Finding found" and "flag captured" are different events.
- **Controlled parallelism** — at most two active challenge environments at a time, in order.
- **Clean runtime isolation** — each challenge in its own Docker network, state reset between attempts, exploitation tools run inside the production `pentest-tools` container.
- **Stop on flag** — dual-engine cross-check is used when the agent is stuck or evidence is ambiguous; once the scorer accepts a captured flag, the lab is solved and budget stops. On the hardest labs, two challenges were cracked only after the second independent engine surfaced an endpoint and gadget the first had missed.

Reporting standard: validate challenge flag delivery before scoring agents; keep raw upstream and patched-validation results separate; publish runtime patches that affect score interpretation; treat flag benchmarks as binary; preserve per-lab techniques so misses become knowledge-pack improvements.
