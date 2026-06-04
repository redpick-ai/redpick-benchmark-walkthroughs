# The 85-to-100 Plateau — Architectural Takeaways

*Expanded from the RedPick blog post [104/104 on XBOW](https://redpick.ai/blog/redpick-scores-100-xbow) §8.*

Building an agent that reaches 85% on XBOW is hard. Getting from 85% to 100% is a different kind of hard — it requires solving the tail of challenges that resist standard approaches. Here is what we learned.

## 1. Multi-step chains are the norm, not the exception

Most XBOW challenges are not single-vulnerability exploits. The typical Level 2 challenge requires:

1. Discover default or weak credentials
2. Authenticate and explore the authenticated attack surface
3. Identify the injection point or logic flaw
4. Craft a payload that bypasses application-specific filters
5. Extract the flag through the confirmed vulnerability

This is a 5-step chain where failure at any step means failure overall.

**Architectural implication**: an agent that can find XSS but cannot first discover that `admin:admin` gets past the login page will score 0 on that benchmark. The agent architecture must support persistent multi-step exploration with inter-step context, not just one-shot vulnerability detection.

## 2. Filter bypass requires reasoning, not fuzzing

The XBOW XSS challenges (23 of them, the largest category) each present a different filter context. Brute-forcing payloads from a wordlist works for some, but the Level 2 and Level 3 challenges require understanding *why* a payload is being blocked and constructing a bypass for *that specific filter*.

**Example**: XBEN-010 (Cross 32, Level 3) uses JavaScript hex escapes in a `document.write()` context — no standard payload list contains the correct bypass because the filter is custom to this benchmark. The agent must analyze the filter's behavior, identify its blind spots, and construct a novel payload.

**Architectural implication**: payload generation must be reasoning-based, not wordlist-based. The agent needs to be able to:
- Observe what a payload does (rendered? escaped? stripped?)
- Infer the filter's transformation rules
- Construct a payload that exploits the inferred rules

## 3. The 85%-to-100% gap is an architectural problem, not a prompting problem

The first 85% of XBOW is solvable with good prompts, solid tool integration, and standard pentesting methodology. The last 15% is not.

**No amount of prompt refinement on a single model will close that gap**, because the remaining challenges require reasoning patterns that the primary model has converged away from.

The 85-to-100 problem is an architectural one. It requires a system design that can bring genuinely different reasoning capabilities to bear when the primary approach is exhausted.

RedPick's multi-model architecture is one solution (see [multi-model-architecture.md](multi-model-architecture.md)). There may be others. But the evidence from XBOW is clear: **single-model agents plateau at ~85%, regardless of how sophisticated the prompting is.**

## 4. Deserialization and protocol-level attacks are the hardest category

Across all 104 benchmarks, the challenges that required the most reasoning effort were:

**Deserialization (6 challenges)**:
- Requires deep language-specific knowledge
- PHP `__wakeup()` chains (Phar)
- Python pickle `__reduce__`
- Java ysoserial gadgets
- .NET BinaryFormatter chains

**Protocol-level attacks (2 challenges)**:
- HTTP request smuggling (CL.TE, TE.CL, TE.TE desync)
- TOCTOU race conditions with tight timing windows

These are also:
- The classes least likely to be found by traditional scanning tools
- The classes where the multi-model architecture proved most valuable
- The classes that most clearly benefit from language-specific security knowledge

**Architectural implication**: deep vulnerability-class-specific knowledge matters. Generic exploit-hunting agents plateau earlier. The agent needs to know how PHP deserialization works, not just "deserialization is a vulnerability."

## Lessons for future agent builders

1. **Don't optimize for the first 85%.** That's a solved problem. Everyone's there.
2. **The tail is where the value is.** The 15% that resist standard approaches are the findings that matter most in real engagements.
3. **Architecture > prompting.** When you're stuck at a plateau, the answer is usually structural (multi-model, verification-first, chain-aware), not prompt-level.
4. **Language-specific depth scales better than generic breadth.** An agent that deeply understands PHP serialization, Python pickle, Java ysoserial will outperform an agent that knows "deserialization is bad."
5. **Evidence matters more than percentages.** A 100% with published Tier-1 evidence for every challenge is much more informative than a 95% with no reproducibility.
