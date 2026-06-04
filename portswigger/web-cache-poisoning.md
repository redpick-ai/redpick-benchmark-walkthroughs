# PortSwigger Academy — Web Cache Poisoning & Deception (18 labs)

*Expanded from the RedPick blog post [100% on PortSwigger Academy](https://redpick.ai/blog/redpick-scores-100-portswigger).*

These 18 labs (13 poisoning + 5 deception) cover the full spectrum of cache-based attacks.

## Web cache poisoning (13 labs)

**Attack flow:**

1. **Identify unkeyed inputs** — headers, cookies, query parameters that the cache ignores but the application processes. Common unkeyed candidates:
   - `X-Forwarded-Host`, `X-Forwarded-Scheme`, `X-Original-URL`
   - `Cookie` (often unkeyed)
   - Custom headers set by frontend framework
   - Parameter pollution (multiple `utm_content` values, one ignored by cache)

2. **Find a gadget** that reflects the unkeyed input into the response:
   - Imported scripts: `<script src="https://{Host}/script.js">`
   - Redirect targets: `Location: https://{X-Forwarded-Host}/path`
   - Link tags: `<link rel="canonical" href="//{unkeyed-input}/path">`
   - Error messages: `Sorry, {user-input} not found` (if cached on error)

3. **Poison a high-traffic cache key** by sending a crafted request with the unkeyed input set to attacker-controlled content.

4. **Wait for the cache hit**: subsequent users requesting the same cache key receive the poisoned response.

A single poisoned response can deliver XSS to every user who hits the cached page — potentially thousands of victims from a single injection.

## Detection heuristics

- **Cache status headers**: `X-Cache`, `CF-Cache-Status`, `Age`, `Cache-Control`
- **Cache key analysis**: test which headers affect cache key by sending same path with different header combinations and observing whether cache HIT occurs
- **Unkeyed reflection**: send unique canary strings in various headers, then fetch the path without the headers and check if canary is present

## Web cache deception (5 labs)

The inverse attack: trick the cache into storing an authenticated user's personalized response under a public cache key.

**Attack flow:**

1. **Craft a URL** that the cache treats as static (`/profile/nonexistent.css`) but the application treats as dynamic (`/profile`)
2. **Convince the victim** to visit the crafted URL (phishing, malicious link, cross-site navigation)
3. **Cache stores** the victim's authenticated response under the static-looking cache key
4. **Attacker fetches** the cached response containing victim's session data, API keys, PII, or internal account details

**Common crafted paths:**

- `/profile/anything.css` — cache serves `.css` cached while app ignores path suffix and serves profile
- `/profile;.css` — semicolons ignored by some servers but kept by cache
- `/profile%00.css` — null byte truncation in some parsers
- `/profile/..%2fanything.css` — encoded path traversal

## Expert-level combinations

**Cache key normalization exploits**: discover that the cache normalizes paths in specific ways (removing trailing slashes, decoding percent-encoding, lowercasing) while the application does not — or vice versa.

**Parameter cloaking**: `utm_content` and other analytics parameters are typically excluded from cache keys. If any of them reach application logic (e.g., reflected in a tracking pixel), they become unkeyed injection points.

**Fat GET requests**: body parameters on GET requests are typically ignored by the cache (it keys on path+query), but some applications process them. This creates an unkeyed injection channel that's particularly sneaky because the body content doesn't appear in cache key or URL.

**Multi-layer poisoning**: CDN + cache + application each have different header handling. What one layer strips, another preserves. The agent must reason about the full chain, not just one cache layer.

## Why this category matters

Cache poisoning and deception are "force multiplier" vulnerabilities:

- **Amplification**: one injection can hit thousands of users
- **Persistence**: poisoning can last for the cache TTL (hours to days)
- **Stealth**: the attacker's IP never touches the victim — victims are compromised purely through cache serving
- **Cross-user impact**: unlike most web vulnerabilities which affect the attacker's own session

These labs tested RedPick's ability to reason about multi-layer HTTP infrastructure, not just application code.

## Tool notes

PortSwigger's own [Param Miner](https://github.com/PortSwigger/param-miner) extension is the standard tool for finding unkeyed headers and parameters. RedPick integrates similar logic into its scanning phase, automatically probing for cache behavior before moving on to gadget-finding.
