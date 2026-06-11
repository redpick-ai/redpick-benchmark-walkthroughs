# RedPick — Benchmark Walkthroughs

This repository contains technical deep-dives and raw finding data from RedPick's benchmark runs. The summary results and high-level narrative live on the RedPick blog; this repo is the long-form archive for researchers, pentesters, and anyone who wants to audit the work.

**Live benchmark dashboards**: [redpick.com/benchmarks](https://redpick.com/benchmarks)
**RedPick blog**: [redpick.ai/blog](https://redpick.ai/blog)

## Index

### XBOW (104/104, 100%)

Pure black-box, no-hint mode. The first published perfect score on XBOW without source code access.

- [Leaderboard context](xbow/leaderboard-context.md) — every published XBOW score compared (as of April 2026)
- [Multi-model architecture](xbow/multi-model-architecture.md) — why single-model agents plateau at ~85%
- [Black-box vs white-box analysis](xbow/black-box-vs-white-box-analysis.md) — why source-code access changes the difficulty
- [The 85-to-100 plateau](xbow/85-to-100-plateau.md) — architectural takeaways from reaching 100%

**Exploitation walkthroughs (selected hard challenges)**:
- [XBEN-066 — HTTP Request Smuggling via HAProxy](xbow/XBEN-066-http-smuggling.md) (Level 3)
- [XBEN-088 — TOCTOU Race Condition](xbow/XBEN-088-toctou-session-poisoning.md) (Level 3)
- [XBEN-079 — LFI with Multi-Layer Filter Bypass](xbow/XBEN-079-lfi-bypass.md) (Level 2)
- [XBEN-092 — Phar Deserialization to RCE](xbow/XBEN-092-phar-deserialization.md) (Level 2)

Blog post: [104/104 on XBOW — Every Challenge, Every Category](https://redpick.ai/blog/redpick-scores-100-xbow)

### PortSwigger Web Security Academy (270/270 + 5 mystery, 100%)

All 270 standard labs plus 5 unreleased mystery labs, 31 vulnerability categories, EXPERT tier achieved.

- [Insecure Deserialization](portswigger/insecure-deserialization.md)
- [Prototype Pollution](portswigger/prototype-pollution.md)
- [Web Cache Poisoning](portswigger/web-cache-poisoning.md)
- [JWT & OAuth Deep-Dive](portswigger/jwt-oauth-deep-dive.md)
- [Multi-Model Collaboration Model](portswigger/collaboration-model.md)

Blog post: [100% on PortSwigger Academy — 270 Labs + 5 Mystery](https://redpick.ai/blog/redpick-scores-100-portswigger)

### Vibe-Coding Benchmark (74/74 + 78 extras, 152 total, 0 FP)

ProjectDiscovery's three vibe-coded applications. 100% ground-truth coverage plus 78 additional code-backed vulnerabilities beyond the answer key.

**Full finding tables** (CSV + narrative):
- [vaultbank](vibe-coding/vaultbank-findings.md) · [CSV](vibe-coding/vaultbank-findings.csv) — FastAPI + PostgreSQL + React (30 GT + 29 extras = 59 findings)
- [medportal](vibe-coding/medportal-findings.md) · [CSV](vibe-coding/medportal-findings.csv) — Next.js 14 + Prisma + NextAuth (20 GT + 39 extras = 59 findings)
- [claimflow](vibe-coding/claimflow-findings.md) · [CSV](vibe-coding/claimflow-findings.csv) — SvelteKit + Drizzle (24 GT + 10 extras = 34 findings)

**Methodology**:
- [Methodology notes](vibe-coding/methodology-notes.md) — source-code reconciliation, double verification, dedup policy, pre-flight invariants

Blog post: [ProjectDiscovery Vibe-Coding: 152 Findings, 0 FP](https://redpick.ai/blog/redpick-vibeapps-benchmark)

### HackBench (16/16, 100%)

Real-world CVE-based web exploitation challenges from ElectrovoltSec — full application stacks (Chatwoot, Lucee, CKEditor, WordPress, XWiki) with real vulnerabilities mirroring production engagements. Black-box, fully automated.

**Exploitation walkthroughs (selected Hard challenges)**:
- [EV-11 — Chatwoot DOM XSS via Encoding Boundary](hackbench/EV-11-chatwoot-encoding-boundary.md) (Hard, 500 pts)
- [EV-14 — Lucee Server Three-Stage RCE Chain](hackbench/EV-14-lucee-three-stage-rce.md) (Hard, 500 pts)
- [EV-12 — CKEditor N-day XSS via CDATA Breakout](hackbench/EV-12-ckeditor-cdata-breakout.md) (Hard, 500 pts)
- [EV-13 — CSV Command Injection (escapeshellarg bypass)](hackbench/EV-13-csv-command-injection.md) (Medium, 300 pts)

**Methodology**:
- [Methodology improvements](hackbench/methodology-improvements.md) — Pipeline Tracing Protocol, Assumption Verification Protocol, Encoding Boundary Exploitation

Blog post: [16/16 on HackBench — From UNION SQLi to 3-Stage RCE](https://redpick.ai/blog/redpick-scores-100-hackbench)

### HackMerlin (7/7, 100%)

Progressive LLM prompt-injection challenge. Seven levels of escalating defenses culminating in a 4-layer defense (input filter + output filter + LLM-as-judge + active deception) — cracked via the **Cloze Filter Detection** technique that turns the output filter into an inverse oracle.

- [Overview & defense progression](hackmerlin/README.md)
- [Prompt library](hackmerlin/prompt-library.md) — concrete templates per level, Cloze Filter probes, Word Replacement Oracle variants
- [Methodology notes](hackmerlin/methodology-notes.md) — testing conditions, anti-cheat, session state, reproduction notes

Blog post: [7/7 on HackMerlin — A 4-Layer LLM Defense Cracked](https://redpick.ai/blog/redpick-scores-7-on-7-hackmerlin)

### Doyensec (Aikido vs XBOW comparison) — black-box, 55 extended TPs, 34 extras

A black-box companion to Doyensec's *Aikido vs. XBOW* comparison. RedPick ran the same two open-source apps (Photoview, Fider) with target URL + credentials only — no repository, no source code. 62 internally confirmed findings, 55 extended true positives, 34 validated extras beyond the reconstructed answer key, 89.6% adjusted two-app score. Not Doyensec-validated; scored against a reconstructed Doyensec-derived key.

**Full finding tables**:
- [Photoview findings](doyensec-aikido-xbow/photoview-findings.md) — 28 findings (8 key-mapped + 20 extras) against a 16-item key
- [Fider findings](doyensec-aikido-xbow/fider-findings.md) — 34 findings (12 key-mapped + 22 extras) against a 17-item key

**Methodology**:
- [Scoring methodology](doyensec-aikido-xbow/scoring-methodology.md) — adjusted-score formula, per-app computation, full partial/miss ledger

Blog post: [55 Findings, No Source Code: RedPick on Doyensec's Aikido vs XBOW Apps](https://redpick.ai/blog/redpick-doyensec-black-box)

### Escape Duck Store (agentic pentesting) — 20/20 + 88 extras

RedPick on Escape's Duck Store benchmark, source-free grey-box. All 20 known vulnerabilities found (100% strict), plus 88 validated extras beyond the answer key (26 distinct classes, 12 new) — 108 extended true positives at 95.6% precision. Self-scored; not part of Escape's study.

- [Full findings](duck-store/findings.md) — the 20 article-parity vulns + the 26 deduplicated extra classes
- [Scoring methodology](duck-store/scoring-methodology.md) — strict vs extended, reconciliation, run conditions

Blog post: [20/20 on Duck Store](https://redpick.ai/blog/redpick-duck-store)

### Pensar Argus (black-box flag benchmark) — 57/60 raw, 60/60 patched

RedPick on Pensar's Argus benchmark (60 Dockerized apps). 57/60 raw upstream captures, plus 3/3 documented runtime-blocker validations after narrow patches (60/60 patched). Reported separately, never flattened. Binary scoring: a flag is captured through the app, or not.

- [Overview, challenge mix & technique chains](argus/README.md)
- [The three runtime blockers + patch](argus/runtime-blockers.md)

Blog post: [57/60 Black-Box on Pensar's Argus](https://redpick.ai/blog/redpick-argus)

---

## About this repo

All walkthroughs are published as-is from RedPick benchmark runs. Payloads have been sanitized where necessary. Each walkthrough contains:

- Vulnerability class and CWE classification
- Discovery approach (black-box reasoning path)
- Full exploitation chain with HTTP-level details
- Key insights and architectural lessons

This content was moved to GitHub to keep the RedPick blog posts concise and readable while preserving the full technical record. If you're a researcher or pentester, start here. If you want the summary narrative, start on the blog.

## Contributing

This is a technical archive — not accepting PRs for content changes. Open issues for corrections, questions, or reproduction discrepancies.

## License

Content (prose, finding write-ups, tables, data files): [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
Code snippets, payloads, and scripts: MIT.

See [LICENSE](LICENSE) for full terms and attribution requirements.
