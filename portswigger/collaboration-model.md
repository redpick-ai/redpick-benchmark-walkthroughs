# PortSwigger Academy — Multi-Model Collaboration Model

*Expanded from the RedPick blog posts [100% on PortSwigger Academy](https://redpick.ai/blog/redpick-scores-100-portswigger) and [104/104 on XBOW](https://redpick.ai/blog/redpick-scores-100-xbow).*

The final set of PortSwigger challenges — particularly the Expert-tier HTTP/2 smuggling labs and custom deserialization chains — were solved using a multi-model architecture.

## The collaboration pattern

The primary AI engine handles reconnaissance, discovery, initial testing, and most exploitation autonomously. When reasoning stalls after exhausting known techniques on a specific challenge, a secondary AI engine is dispatched as an independent attacker to provide a fresh analytical perspective and suggest alternative exploitation paths.

## Where collaboration proves most valuable

**Protocol-level attacks requiring raw byte manipulation**: HTTP/2 frame crafting, custom serialization formats, packet-level timing attacks. The primary engine is trained to think in terms of HTTP libraries; the secondary engine with different priors explores byte-level manipulation.

**Multi-step chains** where the first steps work but the final escalation path is unclear. The primary engine is stuck at step N with a valid primitive but no obvious path to impact. The secondary engine reconsiders step N-2 with a different hypothesis, unlocking a completely different chain.

**Blind/OOB exploitation** where setting up the callback infrastructure and correlating results requires careful orchestration. The secondary engine verifies payload delivery independently, catching cases where the primary engine is confident but the callback never fires.

## Specific PortSwigger labs where collaboration was decisive

1. **HTTP/2 client-side desync (Expert)** — primary engine was constructing H2 frames but missing the exact stream reuse pattern. Secondary engine identified the multiplexing window.

2. **Server-side pause-based smuggling (Expert)** — primary engine ruled out smuggling because timing was inconsistent. Secondary engine pushed further and discovered the pause threshold.

3. **Custom Java deserialization chain (Expert)** — primary engine was trying standard ysoserial gadgets. Secondary engine suggested analyzing the application-specific classes for unique gadget opportunities.

4. **Algorithm confusion in JWT** — primary engine was testing RSA-to-HMAC confusion. Secondary engine suggested testing the inverse (HMAC-to-RSA via `kid` manipulation).

## Not a panacea

Multi-model collaboration adds cost (API calls, token spend, coordination overhead). It's dispatched selectively, not for every challenge. Criteria for dispatch:

- Primary engine has failed N consecutive exploitation attempts
- Primary engine's reasoning has entered a detected loop (same hypotheses repeating)
- The challenge is in a category known to benefit from multi-model (protocol-level, deserialization, complex authz)

For straightforward labs (reflected XSS, standard SQLi, IDOR), single-engine execution is faster and equally effective. Multi-model is the tool for the long tail.

## Architectural philosophy

The collaboration isn't "consensus between models." It's **divergent exploration**. When the primary engine has converged on a wrong hypothesis, the last thing you want is agreement — you want disagreement. The secondary engine's job is to consider what the primary dismissed.

This is why RedPick uses structurally different models (not two instances of the same model). Different training data, different RL objectives, different reasoning priors → genuinely different blind spots.

## See also

- [XBOW multi-model architecture](../xbow/multi-model-architecture.md) — the architectural foundation
- [85-to-100 plateau](../xbow/85-to-100-plateau.md) — why this matters for reaching perfect scores
