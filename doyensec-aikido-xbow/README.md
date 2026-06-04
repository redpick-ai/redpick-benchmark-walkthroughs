# RedPick — Black-box runs on Doyensec's Aikido vs. XBOW applications

Supporting data for the blog post **"55 Findings, No Source Code: RedPick on Doyensec's Aikido vs XBOW Apps"**
(redpick.ai/blog/redpick-doyensec-black-box).

Reference comparison: Doyensec, ["Comparing AI Application Security Testing Platforms: Aikido vs. XBOW"](https://blog.doyensec.com/2026/05/27/aikido-xbow.html) (May 2026).

RedPick ran the same two open-source applications — **Photoview** and **Fider** — in **pure black-box mode**: target URL and credentials only, no repository, no source-code upload, no static review, no answer-key lookup during testing. Because Doyensec published their findings only as a partially redacted spreadsheet, scoring here uses a **reconstructed Doyensec-derived answer key** — one item per distinct vulnerability family described in the public report. These counts are RedPick's own (internal replay + scoring); they are **not** Doyensec-validated.

## Contents

| File | What's in it |
| --- | --- |
| [`photoview-findings.md`](./photoview-findings.md) | Answer-key coverage (16 items) + validated extras for Photoview |
| [`fider-findings.md`](./fider-findings.md) | Answer-key coverage (17 items) + category ledger + validated extras for Fider |
| [`scoring-methodology.md`](./scoring-methodology.md) | The adjusted-score formula, per-app computation, and the full partial/miss ledger |

## Headline numbers (combined, two apps)

| Metric | Photoview | Fider | Combined |
| --- | :---: | :---: | :---: |
| Internally confirmed findings | 28 | 34 | **62** |
| — mapping to key items | 8 | 12 | **20** |
| — extras (outside key) | 20 | 22 | **42** |
| Reconstructed key items | 16 | 17 | **33** |
| Found / partial / missed | 10 / 5 / 1 | 11 / 5 / 1 | **21 / 10 / 2** |
| Strict key score | 12.5 / 16 | 13.0 / 17 | **25.5 / 33 (77.3%)** |
| Validated extras outside the key | 18 | 16 | **34** |
| Adjusted score | 30.5 / 34 (89.7%) | 29.5 / 33 (89.4%) | **60.0 / 67 (89.6%)** |
| Extended true positives | 28 | 27 | **55** |

A note on the bases: *internally confirmed* is a **finding** count (62 = 20 key-mapped + 42 extras); *found / partial / missed* are **key-item** counts (33 reference items); *extended true positives* (55) = the 21 key items found at full credit + the 34 validated extras. One finding can satisfy several key items, so the 20 key-mapped findings cover more than 20 items.

Public Doyensec counts for context: **Aikido 49 TP / 2 FP**, **XBOW 31 TP / 1 FP** — both manually validated by Doyensec, both with source-code access.
