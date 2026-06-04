# PortSwigger Academy — JWT, OAuth, and Authentication (28 labs)

*Expanded from the RedPick blog post [100% on PortSwigger Academy](https://redpick.ai/blog/redpick-scores-100-portswigger).*

28 labs across three authentication-adjacent categories.

## JWT attacks (8 labs)

**None algorithm attack**: if the server accepts `alg: none` JWTs, any signed token can be forged. Strip the signature, set alg to `none`, and submit.

**Weak signing secret**: cracked via hashcat with rockyou.txt or custom wordlists. Common weak secrets: `secret`, `password`, empty string, company name variants. Hashcat mode 16500 for JWT HS256.

**Algorithm confusion (HS256 vs RS256)**: server expects RS256 (asymmetric, public/private key pair). Attacker submits a token signed with HS256 using the server's public key as the HMAC secret — if the library doesn't enforce algorithm matching, the signature validates.

**JWK/JKU header injection**: the JWT header can contain:
- `jwk`: an embedded JSON Web Key used to verify the signature
- `jku`: a URL to fetch the JWK from

If the server blindly trusts these, the attacker can:
- Embed their own key in `jwk` and sign the token with the matching private key
- Point `jku` to an attacker-controlled key server

Both result in forged tokens that pass validation.

**`kid` parameter path traversal**: the `kid` (Key ID) header tells the server which key to use for verification. If the server uses `kid` as a filesystem path or SQL lookup:
- Path traversal: `kid: ../../../../dev/null` → verification against an empty string → any signature matching empty string validates
- SQL injection: `kid: ' UNION SELECT 'known-string'-- ` → inject a known value into the key lookup → sign with that known value

## OAuth (6 labs)

**Redirect URI manipulation**: the OAuth spec requires redirect_uri to match what's registered. But servers often implement this incorrectly:
- **Append paths**: `https://trusted.com/callback` accepts `/callback/attack.html?code=...`
- **Subdomain wildcards**: `*.trusted.com` allows `attacker.trusted.com`
- **Parser inconsistencies**: `https://trusted.com@attacker.com` parsed as `trusted.com` by validation but `attacker.com` by browser
- **URL fragment manipulation**: `https://trusted.com/callback#@attacker.com`

**Implicit grant token theft**: in implicit flow, the access token is in the URL fragment. XSS on the redirect page → steal token via `window.location.hash`.

**Authorization code replay**: if the server doesn't invalidate codes after use (or allows multiple swaps to access tokens), the attacker can replay a captured code.

**Forced profile linking**: attacker triggers OAuth login as themselves, then injects their OAuth identity into the victim's session. The victim's account is now linked to the attacker's Google/GitHub identity — attacker logs in as victim via OAuth.

**CSRF in OAuth flow**: missing `state` parameter validation allows an attacker to force a victim to complete an OAuth flow with attacker's credentials, linking the victim's session to attacker's identity (variant of forced profile linking).

## Authentication (14 labs)

**Username enumeration**: subtle response differences reveal valid usernames:
- Different error messages ("Invalid username" vs "Invalid password")
- Different response times (bcrypt runs for valid usernames, fast-fails for invalid)
- Different rate-limit thresholds
- Different HTTP status codes

**Brute force with IP rotation**: if rate limiting is per-IP, rotate through proxies. If per-user, use password spraying (one guess per user, many users).

**Stay-logged-in cookie forgery**: "Remember me" cookies often contain predictable structures (username + hash of password + IP). If the hash is weak or the IP isn't enforced, forge cookies for other users.

**Offline password cracking**: if the authentication system returns hashes in some form (leaked in responses, included in session tokens), extract and crack offline.

**2FA bypass via response manipulation**: some 2FA implementations return a JSON response `{"success": false, "require2fa": true}`. Tampering the response client-side (in a PoC context) can reveal if the server state was updated despite failure.

**Password reset poisoning**: Host header injection → password reset emails sent with attacker's domain in the reset link → attacker captures the reset token.

## Authentication architectural insights

The most common vulnerability pattern: **trust boundaries mismatched between components.**

- OAuth provider validates, application doesn't re-check
- JWT signature validates, but the `kid` or `iss` isn't validated
- Password hash is secure, but session ID is predictable
- 2FA is enforced on login, but not on sensitive actions (password change, email change)

RedPick's approach focuses on identifying these boundary mismatches by testing each trust relationship independently.
