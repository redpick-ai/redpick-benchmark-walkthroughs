# PortSwigger Academy — Insecure Deserialization (10 labs)

*Expanded from the RedPick blog post [100% on PortSwigger Academy](https://redpick.ai/blog/redpick-scores-100-portswigger).*

The 10 PortSwigger deserialization labs progress from straightforward to deeply technical.

## PHP object injection

Identify serialized PHP objects in cookies (recognizable by `O:` notation — e.g., `O:4:"User":3:{...}`), modify class properties to escalate privileges or trigger magic methods (`__wakeup`, `__destruct`) that reach dangerous sinks. The agent must understand PHP's serialization format well enough to construct valid payloads:

- Adjust property counts when adding/removing fields
- Match string lengths exactly (`s:5:"admin"`)
- Chain magic method invocation through nested objects

## Java deserialization with ysoserial

Recognition signature: magic bytes `AC ED 00 05` (Java serialization stream header). Workflow:

1. Detect serialized object presence (cookies, hidden form fields, HTTP headers)
2. Identify the classpath from error messages (missing class exceptions leak package names)
3. Select the correct ysoserial gadget chain: CommonsCollections1-11, Spring1-2, Hibernate1-2, Jdk7u21, etc.
4. Generate the payload with `java -jar ysoserial.jar <gadget> "<command>"`
5. Base64-encode and submit

Target: achieve remote command execution through the gadget chain invoking `Runtime.exec()` or a similar sink.

## Custom gadget chain construction (Expert)

The hardest deserialization lab provides no pre-built ysoserial chain. The agent must:

1. **Analyze available classes** in the application's classpath (via exposed JARs, error messages, or behavioral fingerprinting)
2. **Trace which methods are invoked during deserialization** — `readObject()`, `validateObject()`, custom `readResolve()` implementations, lifecycle callbacks
3. **Identify reachable sinks** — file I/O, reflection, process spawning, network I/O
4. **Construct a chain of method calls** that reaches a sink with attacker-controlled parameters
5. **Serialize the chain** manually or through tooling like `Serialization Dumper`

This is the deserialization equivalent of writing a custom exploit — no off-the-shelf tools work. It requires deep understanding of Java object lifecycle and the specific application's class hierarchy.

## Key technique: magic method chaining

The art of deserialization exploitation is chaining magic methods across multiple classes to reach a sink. A typical chain:

1. Deserialized class X's `__wakeup()` calls `$this->target->process()`
2. Class Y (in the `target` property) has a `process()` method that uses `$this->data` in a string context
3. Class Z (in the `data` property) has a `__toString()` that calls `eval()` or similar

Each step must:
- Use classes actually available in the application
- Match method signatures expected during deserialization
- Avoid fatal errors that would abort the chain before reaching the sink

## Detection vs exploitation

Detection (finding the vulnerability) is often easier than exploitation (constructing a working chain). Many labs contain a deserialization flaw that can be detected with a single probe:

- Error probe: submit invalid serialized data → observe stack trace
- Timing probe: submit `phar://` or equivalent → measure response delay
- Magic byte probe: look for `AC ED 00 05` in cookies/headers

Full exploitation then requires the work described above.
