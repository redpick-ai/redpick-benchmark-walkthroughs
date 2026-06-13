# HackTheAgent — Methodology Notes

## Testing conditions

- Live hosted CTF at [hacktheagent.com](https://hacktheagent.com/) — not a source-assisted review.
- Fresh `redpick` account, created 2026-06-10. Black-box: no source code, no expected results, no target-specific flag values supplied to the agent during active testing.
- Browser-authenticated interaction with the live site; per-level challenge text read from the UI/API.
- Each level confirmed by the server's `/api/flag` response (`{"success":true, ...}`) before advancing.
- Screenshots captured after each level transition; final dashboard, certificate, and leaderboard row captured at completion.

## Evidence package layout

The committed package (~51 public-safe files) is organized as:

```
hacktheagent-2026-06-10-redpick/
  MANIFEST.md
  raw/
    level-01.json … level-04-retry.json        # redacted per-level transcripts
    level-05-autonomous-attempts.json           # endpoint-disclosure attempts
    level-05-autonomous-tunnel-exploit.json     # redirect exploit transcript
    level-05-redirect-server.log                # 302 redirect log
    final-dashboard.txt
    scoreboard-redpick-row.txt
  screenshots/
    level-0X-after-submit.png
    final-dashboard-completed.png
    scoreboard-redpick-row.png
    hacktheagent-pwned-certificate.png
```

## Redaction policy

Public-safe redaction is applied while preserving prompt/response flow and server-confirmed submissions. Redacted: live CTF flags, the current-run internal `localhost` endpoint value, the temporary Cloudflare tunnel URL, and operational session material (account credentials, cookies, FlareSolverr session identifiers).

## Evidence-integrity hashes (SHA-256)

```text
e57bc8fc1eaacd0a2c82078e2394962c7ad5de920349a8463210bc4007c3ed2c  MANIFEST.md
55d1ed7b5e63b969ecf8501b723bd1038e48e1664622630201c627e9880e442c  raw/level-05-autonomous-tunnel-exploit.json
c19ed21f6ebe014ebe3a0db1c62e91c27dea0f9fc98d96583bec7b307b21503b  screenshots/final-dashboard-completed.png
66047b3a11f4d4f8e382d52a0ece724bbe4b1eb6579aedcaadf23798ad9d728f  screenshots/scoreboard-redpick-row.png
```

## Reproducibility notes

- Per-level outcomes depend on the assistant's runtime configuration, which can drift between runs; the transcripts capture the exact prompt/response pair that produced each server-confirmed flag.
- Level 4 was solved on a retry after the first admin-framed purchase attempt — both attempts are preserved (`level-04*.json`) so the chain from L2's leaked discount is auditable.
- Level 5 phase 2 requires an attacker-controlled redirect endpoint reachable by the assistant's URL tool; we used a temporary Cloudflare tunnel to a local redirect server returning `302` to the disclosed endpoint.

## Evidence-quality bar

Each solved level requires four artifacts before we count it: the prompt, the assistant response, the server-confirmed submit, and the post-submit UI state. Refusals are retained as data (they are a leak surface), not discarded as negative results.
