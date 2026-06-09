# RedPick on Escape's Duck Store

Supporting data for the blog post **"20/20 on Duck Store: RedPick on Escape's Agentic Pentesting Benchmark"** (redpick.ai/blog/redpick-duck-store).

Reference: Escape, ["Benchmarking AI Pentesting Tools: A Practical Comparison"](https://escape.tech/blog/benchmarking-agentic-ai-pentesting-tools/) (2026-04-30). Duck Store is a deliberately vulnerable FastAPI + React e-commerce app with a documented REST API.

RedPick ran the same target under **article-parity, source-free grey-box** conditions: URL + `/openapi.json` pointer + default credentials only, no source code, no answer-key access during testing. RedPick was **not** part of Escape's study and Escape did not validate these results — the counts are RedPick's own scorer output.

## Result

| Metric | Value |
| --- | :---: |
| Known vulnerabilities (article-parity key) | 20 |
| Found / Partial / Missed | **20 / 0 / 0** |
| Strict key score | **100.0%** |
| Validated extras beyond the key | **88** (26 distinct classes, 12 new) |
| Extended true positives | **108** |
| False positives | 5 |
| Extended precision | **95.6%** |
| Runtime | ~7h |

For context, Escape's published counts on the same 20-item key: Escape (multi-model) 15/20, Claude Code (Opus 4.6, raw baseline) 14/20, PentAGI 9/20, Shannon 6/20, Strix 1/20.

## Contents

| File | What's in it |
| --- | --- |
| [`findings.md`](./findings.md) | The 20 article-parity vulnerabilities (CVSS/CWE/OWASP/endpoint) + the 26 deduplicated extra classes |
| [`scoring-methodology.md`](./scoring-methodology.md) | Strict vs extended scoring, reconciliation, run conditions |
