# Multi-Model Architecture — Why Single AI Models Plateau at 85%

*Expanded from the RedPick blog post [104/104 on XBOW](https://redpick.ai/blog/redpick-scores-100-xbow) §5.*

## The 85% ceiling

The first ~85% of XBOW benchmarks fell to our primary engine's autonomous testing. That baseline is comparable to XBOW's own 85% and Cyber-AutoAgent's 84.62% — it represents the natural ceiling of what a single-model agent can achieve with standard pentesting methodology.

The remaining ~15% required something different.

## What breaks single-model agents

These are the challenges where the primary engine has exhausted every technique in its arsenal and is stuck in a reasoning loop — revisiting the same approaches without finding a new angle. The vulnerability classes that consistently resist single-model approaches:

- **HTTP request smuggling** (protocol-level byte manipulation)
- **TOCTOU race conditions** (concurrency semantics)
- **Multi-layer filter bypasses** (string processing order analysis)
- **Complex deserialization chains** (language-specific internals)

Each of these requires reasoning patterns that a single model tends to converge away from. When the primary model latches onto a hypothesis and the hypothesis is wrong, it will keep refining variations of the wrong hypothesis rather than stepping back and considering a fundamentally different angle.

## Multi-model architecture

Our solution is a **multi-model architecture** that combines AI models with genuinely different reasoning patterns. The key insight — validated by the XBOW results — is that different models have different blind spots.

When one engine has converged away from the correct approach, another engine with different reasoning tendencies can identify angles that the first systematically overlooks. This is not about running the same model twice; it is about leveraging complementary strengths.

## Where multi-model proved decisive

The challenges where the multi-model approach was critical:

- **XBEN-066** (HTTP Request Smuggling) — [walkthrough](XBEN-066-http-smuggling.md)
- **XBEN-088** (TOCTOU Race Condition) — [walkthrough](XBEN-088-toctou-session-poisoning.md)
- **XBEN-079** (LFI Multi-Layer Filter Bypass) — [walkthrough](XBEN-079-lfi-bypass.md)
- **XBEN-092** (Phar Deserialization) — [walkthrough](XBEN-092-phar-deserialization.md)

These are exactly the ones that represent the hardest vulnerability classes: protocol-level byte manipulation, concurrency semantics, string processing order analysis, and language-specific internals chains. These are the classes that separate 85% agents from 100% agents.

## Redundant vs complementary parallelism

Cyber-AutoAgent's blog post argues against multi-agent architectures due to coordination overhead and strategic incoherence. We agree that *redundant* parallelism — running copies of the same model on the same problem — adds cost without value.

But *complementary* multi-model collaboration, where different models contribute different reasoning capabilities, is the architectural choice that closes the gap between 85% and 100%. The XBOW results are the evidence.

## Technical implementation notes

Multi-model collaboration in RedPick is not simply "run model A, if stuck run model B". The architecture includes:

- **Explicit stuck-detection**: primary engine detects reasoning loops (repeated hypotheses, declining progress on action success rates)
- **Secondary engine with different reasoning style**: not a second instance of the same model
- **Shared context with divergent exploration**: the secondary engine inherits what the primary has tried, but explores the remaining space with different priors
- **Cross-validation on promising paths**: when either engine finds a candidate exploit, the other verifies it independently before the agent commits

This avoids the coordination overhead pitfall while capturing the diversity benefit.

## Further reading

- [85-to-100-plateau.md](85-to-100-plateau.md) — architectural takeaways
- [black-box-vs-white-box-analysis.md](black-box-vs-white-box-analysis.md) — how source-code access changes difficulty
