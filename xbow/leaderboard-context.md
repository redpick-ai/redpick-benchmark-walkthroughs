# XBOW Benchmark — Leaderboard Context

*Expanded from the RedPick blog post [104/104 on XBOW](https://redpick.ai/blog/redpick-scores-100-xbow) §2. As of April 2026.*

The XBOW benchmark has become the de-facto evaluation standard for AI pentesting agents, and a growing number of teams have published their results. Here is every published score we are aware of — agents, humans, and research prototypes alike.

## The full leaderboard

| Rank | Agent / Team | Score | % | Methodology | Source code | Hints | Source |
|:----:|--------------|:-----:|:-:|:-----------:|:-----------:|:-----:|--------|
| **1** | **RedPick** | **104/104** | **100%** | **Black-box** | **None** | **None (no-hint)** | [Blog post](https://redpick.ai/blog/redpick-scores-100-xbow) |
| 2 | Shannon Lite | 100/104 | 96.15% | **White-box** | **Full source** | None | [GitHub](https://github.com/KeygraphHQ/shannon) |
| 3 | RedVeil | ~92/104 | ~88.5% | Black-box | None | With hints | [redveil.ai](https://www.redveil.ai/additional-resources/comparisons/xbow-alternative) |
| 4 | PentestGPT v2 | ~95/104 | 91% peak | Black-box | None | With hints | [arXiv 2602.17622](https://arxiv.org/abs/2602.17622) |
| 5 | LuaN1aoAgent | ~94/104 | 90.4% | Black-box | None | With hints | [GitHub](https://github.com/SanMuzZzZz/LuaN1aoAgent) |
| — | *Human team (5 pentesters)* | *~91/104* | *87.5%* | *Manual* | *None* | *With hints* | *[XBOW blog](https://xbow.com/blog/xbow-vs-humans)* |
| 6 | Red-MIRROR | ~89/104 | 86% | Black-box | None | With hints | [arXiv 2603.27127](https://arxiv.org/abs/2603.27127) |
| — | *XBOW (own system)* | *~88/104* | *85%* | *Black-box* | *None* | *Own benchmark* | *[XBOW blog](https://xbow.com/blog/benchmarks)* |
| — | *Human principal pentester* | *~88/104* | *85%* | *Manual* | *None* | *With hints* | *[XBOW blog](https://xbow.com/blog/xbow-vs-humans)* |
| 7 | Cyber-AutoAgent | 88/104 | 84.62% | Black-box | None | With hints | [GitHub](https://github.com/westonbrown/Cyber-AutoAgent) |
| 8 | Deadend CLI | 83/104 | 80% | Black-box | None | No hints | [GitHub](https://github.com/xoxruns/deadend-cli) |
| 9 | MAPTA | 80/104 | 76.9% | Black-box | None | With hints | [arXiv 2508.20816](https://arxiv.org/abs/2508.20816) |
| — | *Human staff pentester* | *~61/104* | *59%* | *Manual* | *None* | *With hints* | *[XBOW blog](https://xbow.com/blog/xbow-vs-humans)* |
| 10 | PentestAgent | ~52/104 | 50% | Black-box | None | With hints | [arXiv 2603.27127](https://arxiv.org/abs/2603.27127) |
| 11 | AutoPT | ~48/104 | 46% | Black-box | None | With hints | [arXiv 2603.27127](https://arxiv.org/abs/2603.27127) |

*Italic rows = humans or benchmark creators, for reference. Scores marked ~ are derived from published percentages.*

## What this table tells us

**Three clusters are visible.** The first cluster (46-62%) includes early-generation agents and junior-to-mid pentesters. The second cluster (77-91%) includes mature agents and senior pentesters — all with hints, most plateauing around 85-91%. The third cluster is two entries: Shannon Lite at 96.15% with source code, and RedPick at 100% without it.

**No black-box agent other than RedPick has broken 91%.** PentestGPT v2 reports a 91% peak with hints — that is the highest published black-box result before ours. Every agent above 91% either had source code access (Shannon) or is RedPick.

**Hints matter.** Deadend CLI tested without hints and scored 80%. Most agents that score 85%+ used the standard XBOW repository with full vulnerability descriptions, category tags, and difficulty levels. Our 100% was achieved in no-hint mode — a harder test than what the 85-91% cluster ran against.

## XBOW (the company) — 85% on their own benchmark, with full hints

XBOW's own proprietary system scores approximately **85% (88/104)**. In their blog post, they describe this as *"equivalent to what an experienced pentester could achieve within a week."* XBOW also published [human pentester baselines](https://xbow.com/blog/xbow-vs-humans): a team of five pentesters (junior through principal level) collectively reached 87.5%, with the most experienced individual pentester matching XBOW's own system at 85%.

It is important to note what "their own benchmark" means here: XBOW created these challenges, they know the vulnerability classes, and their system runs against the standard benchmark metadata — full vulnerability descriptions, category tags, difficulty levels. This is the most favorable possible testing condition: you built the challenges, you know what categories they cover, and your agent receives metadata about each one before testing begins. Even under these conditions, their system misses approximately 16 out of 104.

We exceeded this by 16 challenges — and we did it with less information (no hints, no source code) than the benchmark creators themselves used.

## Shannon Lite (96.15% — white-box with source code)

Keygraph's Shannon Lite achieved the highest previously published score: **100/104 (96.15%)**. However, there is a critical methodological difference: Shannon is a **white-box** agent. Their README states explicitly: *"White-box only. Shannon Lite is designed for white-box (source-available) application security testing. It expects access to your application's source code."*

Shannon's XBOW run used a *"hint-free, source-aware variant"* — no XBOW vulnerability descriptions, but full source code of each benchmark application. This is a fundamentally different testing paradigm. A white-box pentester reading the source code can see the SQL query construction, the template rendering call, the deserialization entry point. A black-box pentester must discover these through probing, fuzzing, and behavioral analysis of the live application.

Both approaches are valid and valuable in real-world security testing. But they are not comparable on the same scale. White-box testing with source access is structurally easier than black-box testing without it — that is why organizations do both.

See [black-box-vs-white-box-analysis.md](black-box-vs-white-box-analysis.md) for a detailed breakdown.

## The 85-91% cluster — mature agents with hints

PentestGPT v2 (91% peak), LuaN1aoAgent (90.4%), RedVeil (~88.5%), Red-MIRROR (86%), XBOW (85%), and Cyber-AutoAgent (84.62%) all cluster in the 85-91% range. All used the standard XBOW benchmark with full hints. This cluster represents the current ceiling for single-model black-box agents operating with hint metadata — a strong result, but one that leaves 9-16 challenges unsolved.

For the architectural reasons behind this plateau, see [85-to-100-plateau.md](85-to-100-plateau.md) and [multi-model-architecture.md](multi-model-architecture.md).

## RedPick (100% — pure black-box, no-hint)

Our result: **104/104 (100%)**, the hardest possible testing configuration:

- **Pure black-box**: no source code access at any point during testing. Not even partial source access, not "white-box for discovery then black-box for exploitation" — zero source code, zero configuration files, zero access to the target's filesystem. Every vulnerability was discovered and exploited exclusively through the application's network attack surface.
- **No-hint mode**: no vulnerability descriptions, no category tags, no difficulty levels. The agent received a benchmark ID and a URL. Nothing else.
- **No answer keys**: no prior knowledge of expected vulnerabilities, flags, or exploitation paths.

This is the first published perfect score on the complete XBOW benchmark set. It is also — to our knowledge — the only 100% achieved through pure black-box testing without any source code access. The next-best black-box result (PentestGPT v2 at 91% peak) was achieved with full hints — we scored 9 percentage points higher with no hints at all.
