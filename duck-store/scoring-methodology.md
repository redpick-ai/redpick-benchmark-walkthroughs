# Duck Store — scoring methodology

How RedPick's source-free grey-box run on Escape's Duck Store was scored. Two views are reported: a **strict article-key score** (for head-to-head comparability with Escape's published numbers) and an **extended scorer view** (for real-pentest signal beyond the finite key).

## Strict article-key score

The article-parity answer key is the 20 vulnerabilities Escape scored. Each is found, partial, or missed.

| Result | Count |
| --- | :---: |
| Known vulnerabilities in the article-parity key | 20 |
| Found | 20 |
| Partial | 0 |
| Missed | 0 |
| Strict score | **20.0 / 20 = 100.0%** |
| Runtime | ~7h |

This is the fair head-to-head measurement: it matches the Escape article exactly. RedPick hit all 20.

## Extended scorer view

The testing agent never reads the answer key; scoring is a separate, post-run operation on frozen evidence.

| Metric | Value |
| --- | :---: |
| Total finding files | 114 |
| Extra findings beyond the 20-item key | 94 |
| Validated extras credited by the scorer | 88 |
| Extended true positives | 108 |
| False positives | 5 |
| Near-duplicates | 1 |
| Extended precision | 95.6% |
| Adjusted score | 108.0 / 108 = 100.0% |

Reconciliation:
- 114 total files = 108 extended TPs + 5 FPs + 1 near-duplicate.
- 94 extra-beyond-key = 88 validated extras + 5 FPs + 1 near-duplicate.
- 108 extended TPs = 20 key items + 88 validated extras.
- Extended precision 95.6% = 108 / (108 + 5).

The 88 credited files deduplicate to **26 distinct classes** (see [`findings.md`](./findings.md)); ~49 files are deeper instances of the 20 scored items and 12 classes are new beyond the key.

An "extended recall" of 540% appears in the raw scorer output. It is intentionally strange-looking — it only means RedPick did not stop at the finite 20-item key. We do **not** compare it to Escape's 20-item table; it is a different measurement. The stricter, comparable number is the simple one: **20/20 known items found**.

## Why two numbers

- **Strict key score (20/20):** whether the benchmark was solved.
- **Extended view (108 validated TPs, 95.6% precision):** whether the system kept finding useful attack surface after the answer key was exhausted — the real-pentest signal. Real targets expose multiple vulnerable shapes for the same class (e.g. two SSRF routes with different auth and impact; many independently exploitable object/function-authorization failures), which a finite key cannot capture.

## Run conditions (for reproducibility)

- Article-parity grey-box inputs: URL, `/openapi.json`, `/api/`, `admin/admin`, `user/password`. No source code; no answer-key access during testing; no prior RedPick findings read during the run.
- The low-privilege `user/password` account was registered and verified before the run (not consistently present on the hosted instance), then used as the article intended.
- Dual-engine workflow: a primary engine drives testing; a second independent engine re-checks high-impact findings as a peer; automatic re-runs before closure when a pass did not complete cleanly; scoring performed separately after evidence was frozen.
- The current public Duck Store target exposes a broader vulnerability page than the article's 20-item score (including extra MCP items); the article-parity key intentionally excludes those for comparability.
