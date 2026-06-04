# Black-box vs White-box — Why This Is Not a Minor Distinction

*Expanded from the RedPick blog post [104/104 on XBOW](https://redpick.ai/blog/redpick-scores-100-xbow) §6.*

Shannon Lite's 96.15% (100/104) is an impressive result — building an agent that can solve 100 of these benchmarks is a significant engineering achievement. But every comparison of XBOW scores must start with a fundamental question: **did the agent have access to the source code?**

Shannon did. RedPick did not. This is not a minor methodological footnote — it is the difference between two fundamentally different security testing paradigms.

## What source code access means in practice

Consider **XBEN-092** (Phar deserialization). Here is what each approach looks like:

### With source code (Shannon's approach)

The agent reads the PHP class files:
- Sees `ReadClass` with `eval($this->data)` in the `__wakeup()` magic method
- Sees the `phar://` stream wrapper usage in `read_sku.php`
- Sees the base64 upload handler in `sku_process.php`

The vulnerability is visible in the code. The agent needs to construct the exploit payload, but it already knows exactly what to target and why.

### Without source code (RedPick's approach)

The agent sends HTTP requests to a black box:
- Must discover that `/skumodel-srv/` exists through directory fuzzing
- Must infer from application behavior that deserialization is involved
- Must guess the class structure, the magic method chain, and the upload mechanism — all through behavioral probing, error message analysis, and iterative payload construction

The vulnerability is invisible until the agent reasons its way to it.

## The pattern repeats across the benchmark

For **every SSTI challenge**:
- **White-box agent**: sees the template engine and the injection point in the code
- **Black-box agent**: injects canary strings, observes how they render, deduces the template engine from output patterns (different templating languages have distinct escape syntax)

For **every SQLi challenge**:
- **White-box agent**: sees the query construction, knows the database driver, the parameter positions
- **Black-box agent**: infers the database type (MySQL vs PostgreSQL vs MSSQL vs SQLite from error messages and timing), the query structure, and the injection context from response differences

For **every deserialization challenge**:
- **White-box agent**: reads class definitions, identifies gadget chains, constructs payloads directly
- **Black-box agent**: probes for deserialization through `O:`, `a:`, `i:` format injection, infers class presence from error messages, guesses gadget chains

## The concrete impact on scoring

The difference is not theoretical. Consider the testing difficulty gradient:

| Information available | Difficulty | Analogy |
|-----------------------|:----------:|---------|
| Source code + hints + tags | Easiest | Open-book exam with answer guide |
| Source code, no hints | Easy | Open-book exam |
| Hints + tags, no source | Medium | Closed-book with topic list |
| **URL only (RedPick's approach)** | **Hardest** | **Closed-book, no topic list** |

- **Shannon** tested at the "Easy" level — source code, no XBOW hints
- **Cyber-AutoAgent** tested at the "Medium" level — XBOW hints and tags, no source code
- **RedPick** tested at the **"Hardest" level** — no source code, no hints, no tags, no descriptions. Just a URL.

**The result: RedPick scored higher (104/104) with less information (URL only) than every other published agent that tested with more information.** Shannon had full source access and scored 100/104. Cyber-AutoAgent had vulnerability descriptions and scored 88/104. RedPick had nothing but a URL and scored 104/104.

## Not a knock on other projects

This is not a knock on Shannon or Cyber-AutoAgent. Both projects advanced the field and published their results honestly, with clear methodology disclosure. Shannon's README transparently states that it is a white-box tool — there is no attempt to mislead.

But when evaluating claims about XBOW benchmark scores, **the testing methodology must be the first thing you check**. A 96% with source code and a 100% without source code are not comparable numbers — the 100% was achieved under significantly harder conditions.

## Why RedPick chose pure black-box

RedPick could have given itself source code access. The platform supports both white-box code review and black-box dynamic testing — and in real-world engagements, typically runs both. Pure black-box for the XBOW evaluation was chosen because:

1. **It is the realistic scenario.** In most penetration tests, and in all bug bounty programs, the tester does not have access to source code. A tool that performs at 100% in black-box mode will also perform at 100% with source code. The reverse is not guaranteed.

2. **It is the honest comparison.** XBOW designed their benchmarks as black-box challenges. Testing them with source code changes what is being measured.

3. **It is the harder test.** If the goal is to demonstrate capability, the hardest test is the most convincing one.

## Both have a place

To be clear: both testing paradigms have value in real-world security programs.

**White-box** is faster, more thorough for known vulnerability classes, and better suited for pre-release code review. When you have source access (internal applications, CI/CD pipelines, dedicated pentests with customer cooperation), you should use it.

**Black-box** is the realistic adversary model for production applications facing external attackers. It is also the only option for:
- Bug bounty programs
- Third-party assessments where source is not available
- Continuous security testing in production
- Validating defensive controls against the attacks they'll actually face

RedPick supports both modes. The XBOW evaluation specifically chose black-box because that's the hardest test — and therefore the most convincing demonstration of capability.
