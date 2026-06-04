# HackBench — Methodology Improvements

*Expanded from the RedPick blog post [16/16 on HackBench](https://redpick.ai/blog/redpick-scores-100-hackbench).*

Three generalizable techniques were extracted from the HackBench evaluation and integrated into RedPick's production platform. Each emerged from a specific challenge and has been formalized into a reusable protocol.

---

## 1. Pipeline Tracing Protocol

**Originated from**: EV-11 (Chatwoot DOM XSS via encoding boundary)

Before crafting injection payloads, the agent maps the full data transformation pipeline the input will traverse. The pipeline covers every stage from input to sink:

```
Input → Storage → Server Renderer → HTTP Transport → Browser DOM → Client JS → Sink
```

Each stage is tested independently with canary values. Canaries are designed to detect specific transformations:

- `"<>` → detects HTML entity encoding (becomes `&quot;&lt;&gt;`)
- `%22%3C%3E` → detects URL encoding normalization
- `\u0022\u003c\u003e` → detects Unicode escape handling
- `&amp;lt;` → detects double-encoding or decoding

The payload must survive ALL stages, not just work at the sink. This protocol was formalized after EV-11, where the payload had to navigate:

1. Markdown renderer (stage that inerted raw `<`)
2. HTML entity preservation (stage that kept `&lt;` verbatim)
3. DOM parsing (stage that rendered entities as text)
4. `innerText` extraction (stage that decoded entities back to `<`)
5. `innerHTML` re-injection (stage where the decoded `<` became active HTML)

Each stage has different encoding semantics. A payload that is active at stage 1 would be stripped; a payload that is inert through stage 3 but active at stage 5 is the one that fires.

### When this protocol fires

Any finding involving user input reaching a DOM or HTML context through more than one processing step. The agent's XSS playbook now starts with "map the pipeline" before "select a payload."

### Observable impact

In pre-protocol runs, XSS agents default to top-N payload lists and move on when nothing fires. In post-protocol runs, they identify encoding-inert corridors that no payload list contains. EV-11 would have been unsolvable by a pattern-matching approach.

---

## 2. Assumption Verification Protocol

**Originated from**: EV-14 (Lucee three-stage RCE chain)

When executing a multi-step exploit plan — whether from CVE research, AI-assisted analysis, or reconnaissance — verify each assumption **independently** before combining.

The protocol defines three questions per stage:

1. **Injection point**: does input actually reach the sink I'm assuming it reaches?
2. **Processing**: what transformations happen along the path? (See [Pipeline Tracing](#1-pipeline-tracing-protocol))
3. **Trigger**: does the vulnerable code path execute under the observed conditions?

Only craft the full payload after all individual assumptions are confirmed.

This protocol was formalized after EV-14, where each stage of the three-stage RCE chain required independent validation:

- Stage 1 validation: "does `imgProcess.cfm` actually accept unauthenticated writes?" → test with innocuous `.txt` content
- Stage 2 validation: "does the web mapping admin interface actually accept arbitrary virtual paths?" → create a mapping to a known-safe directory and confirm resolution
- Stage 3 validation: "does CFML in a mapped path actually execute?" → serve a harmless CFML file that echoes a constant

Only after all three stages were independently validated did the agent assemble the full chain with the flag-reading CFML payload. Each validation attempt failing would immediately redirect the plan, rather than producing a fully-constructed payload that silently fails at an unknown stage.

### When this protocol fires

Any multi-step exploit chain. The agent will not attempt combined exploitation until each stage is independently reproducible.

### Observable impact

Pre-protocol: multi-stage chains frequently failed with ambiguous symptoms — a 200 response with no flag means one of three stages broke, and debugging required bisecting. Post-protocol: chain failures identify the specific stage that did not meet its precondition.

---

## 3. Encoding Boundary Exploitation

**Originated from**: EV-11 (Chatwoot DOM XSS), generalized into the XSS knowledge pack

A new technique added to the XSS knowledge pack:

> Find encodings that are **inert at intermediate processing stages** but become **active at the final injection point**.

The canonical example from EV-11:

- HTML entities (`&lt;`, `&gt;`, `&quot;`) are inert through markdown renderers because they are not treated as tags
- `innerText` decodes them (this is standard DOM behaviour, not a bug)
- When the decoded text is subsequently injected via `innerHTML`, the entities become active HTML

### Pattern catalog

The technique generalizes to any multi-stage processing pipeline where different stages have different encoding semantics. Known variants now tested:

| Intermediate form | Inert in | Active in |
|-------------------|----------|-----------|
| HTML entities (`&lt;`) | Markdown, plain-text renderers | `innerHTML` after `innerText` decode |
| Unicode escapes (`\u003c`) | JSON parsing | JS `eval` / template string |
| URL encoding (`%3C`) | HTTP transport | Server-side decode before HTML render |
| Base64 (`PHNjcmlwdD4=`) | String comparison / validation | After `atob` / decode step |
| UTF-7 (`+ADw-`) | Standard UTF-8 parsers | Browser with UTF-7 fallback |
| Homoglyphs (`＜`, `❮`) | Keyword-based WAF | Normalization before render |
| Double encoding (`&amp;lt;`) | Single-pass decoder | Multi-pass decoder |

### When this protocol fires

Any finding involving text rendered through multiple stages — particularly when the final sink is `innerHTML`, `document.write`, `eval`, `setTimeout(string, ...)`, or `Function(string)()`. The agent probes for encoding-inert corridors before resorting to payload lists.

### Observable impact

Encoding-boundary bugs are systematically missed by signature-based DAST. They also tend to be missed by single-model LLM agents that default to canonical payload forms. The technique reliably surfaces vulnerabilities in AI-generated codebases where developers have layered "safe" APIs (markdown, JSX, React) without realizing the interactions between them create new sinks.

---

## How these integrate into RedPick

Each of the three protocols is now part of RedPick's generalized testing playbook — not as a hardcoded rule, but as a reasoning pattern the agents use when planning attacks. The protocols are model-agnostic: they describe **how to think about** a class of vulnerability, not a specific payload.

The HackBench evaluation was the forcing function for formalizing them. Subsequent benchmark runs (PortSwigger, XBOW, Vibe-Coding) and production engagements have benefited from the same pipeline tracing, assumption verification, and encoding boundary reasoning — even though those benchmarks test different vulnerability classes.

This is what we mean by "the benchmark is a validation, not an end point" — each benchmark that exposes a reasoning gap becomes a permanent improvement to the production platform.

Blog post: [16/16 on HackBench](https://redpick.ai/blog/redpick-scores-100-hackbench)
