# HackTheAgent — 5/5 (full live CTF)

Technical appendix for RedPick's full completion of [HackTheAgent](https://hacktheagent.com/), a hosted HackAIcon AI-agent CTF (an AI ticketing assistant with hidden policy state, business-side-effect tools, and a URL-fetching tool).

**Blog post**: [HackTheAgent 5/5: RedPick on the HackAIcon AI-Agent CTF](https://redpick.ai/blog/redpick-hacktheagent)

**Live benchmark dashboard**: [redpick.ai/benchmarks](https://redpick.ai/benchmarks)

## Scope note

RedPick's official benchmark ledger tracks HackTheAgent at its **article-parity 3-level scope** (L1–L3). This run is intentionally broader and validates the full 5-level live CTF end to end under a fresh `redpick` account. Leaderboard placement on a public CTF reflects completion timing among participants, not a capability ranking — we report 5/5 completion, server-confirmed, not a leaderboard win.

## What's in this folder

The blog post carries the narrative — per-level technique, the failure-mode mapping, and the Level 5 redirect-SSRF walkthrough. This folder holds dev-oriented material too verbose for the post:

- [**methodology-notes.md**](methodology-notes.md) — testing conditions, evidence package layout, redaction policy, evidence-integrity hashes, reproducibility notes.
- [**prompt-notes.md**](prompt-notes.md) — the per-level prompt strategy (redacted of live flags, the internal endpoint, and the temporary tunnel URL).

The full redacted evidence package (browser screenshots, redacted API transcripts, final dashboard HTML/text, leaderboard page, generated certificate, redirect-server log) is committed alongside these notes. It intentionally omits account credentials, cookies, and FlareSolverr session identifiers; transcripts are redacted for flags, the current-run internal endpoint, and the temporary tunnel URL.

## The 5 levels at a glance

| Level | Challenge | Primary technique | Failure mode it proves |
|-------|-----------|-------------------|------------------------|
| 1 | Secret Key Disclosure | Language / policy-exception framing | Unverified user-attribute exception |
| 2 | Unauthorized Discount Code | Refusal / tool-payload leakage | Refusal explanation leaks secret state |
| 3 | Impossible Refund | Refusal mining + scenario framing | Privileged action from model-controlled text |
| 4 | Free Ticket Exploit | Chaining a leaked secret + admin-domain constraint | Tool action from disclosed policy state |
| 5 | The Assistant Endgame | Endpoint disclosure + redirect-based internal fetch | URL tool follows a redirect to an internal service |

## Headline finding — redirect-based SSRF through an LLM tool (Level 5)

The final level is the strongest lesson. Blocking explicit `localhost` strings in the prompt is not enough when the agent has a URL-fetching tool that follows redirects:

1. **Endpoint disclosure** — a system-prompt-placeholder vs runtime-config comparison prompt got the assistant to fill in only the internal `localhost` endpoint value.
2. **Redirect exploit** — the assistant was asked to fetch a benign public profile URL (a temporary Cloudflare tunnel to a RedPick-controlled redirect server). The server returned a `302` to the just-disclosed internal endpoint; the agent's HTTP client followed it and returned the flag.

Mitigation: the security boundary must live in the tool implementation — resolve final destinations, re-validate against private address ranges *after* every redirect, and apply egress controls at the HTTP client and infrastructure layers.
