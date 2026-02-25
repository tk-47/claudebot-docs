# Security Audit Guide for Claudebot

> **What this is:** A prompt you can give Claude Code to perform a full security audit of your Claudebot deployment. Copy the audit prompt below, paste it into Claude Code from your project root, and work through the findings together.

## Before You Start

- Run the audit from your project root directory (where `src/`, `.env`, etc. live)
- Have your `.env` file present (Claude Code will check it but won't leak values)
- The audit covers both local mode (`bot.ts`) and VPS mode (`vps-gateway.ts`)
- Claude Code will read your code, identify vulnerabilities, and help you fix them

---

## The Audit Prompt

Copy everything below the line and paste it into Claude Code:

---

```
Perform a full security audit of this Claudebot codebase. Read every file in src/ and check for the following categories of vulnerabilities. For each issue found, assign a priority (P0 = critical, P1 = high, P2 = medium, P3 = low), explain the risk, and propose a specific fix.

## 1. Subprocess & Command Injection

- Check how Claude Code subprocesses are spawned (look in src/lib/claude.ts or similar)
- Is `--dangerously-skip-permissions` used? If so, this is P0 — any prompt injection in untrusted input (email subjects, calendar events, filenames, webhook payloads) can execute arbitrary commands
- Fix: Replace with `--allowedTools` using a minimal read-only set (Read, Glob, Grep, WebSearch, WebFetch). The bot's direct API integrations handle external services — Claude subprocess only needs conversational AI capabilities
- Check for any Bash/exec/spawn calls that interpolate user input without sanitization

## 2. Authentication & Authorization

- Check every HTTP endpoint in the VPS gateway for authentication
  - Webhook endpoints (Telegram, GitHub deploy, ElevenLabs, Fireflies, etc.) — do they verify signatures/secrets?
  - Data endpoints (health, context, status) — do any leak sensitive info without auth?
- Check webhook signature verification:
  - Is verification skipped when the secret env var is missing? (It should reject, not allow)
  - Are secrets compared with constant-time comparison (`crypto.timingSafeEqual`)? String `===` is vulnerable to timing attacks
- If Microsoft Teams is integrated, check JWT verification:
  - Are all JWT issuers validated? Watch for "fallback" logic that accepts tokens without issuer checks
  - Is the audience claim verified?

## 3. CORS & HTTP Headers

- Check for `Access-Control-Allow-Origin: *` on any endpoint
  - Server-to-server endpoints (Telegram webhooks, GitHub webhooks) don't need CORS at all
  - If a web UI exists, CORS should specify exact origins, never wildcard
- Check for missing security headers if any endpoint serves HTML

## 4. Secrets Management

- Check `.env` and `.env.example` — are any real keys committed?
- Check git history: `git log --all --oneline -- .env* *.env` for accidental commits
- Are API keys passed in URL query parameters? (They appear in server logs, proxy logs, error messages) — they should be in headers instead
- Check for hardcoded secrets, API keys, spreadsheet IDs, or credentials anywhere in source code — these should be env vars
- Are there any console.log statements that could print secrets?

## 5. Input Validation & Path Traversal

- Check file upload handling — are filenames sanitized? Can a crafted filename like `../../.env` escape the upload directory?
- Check any user input that flows into file paths, database queries, or API calls
- If using SQL/Supabase: check for unsanitized LIKE/ilike patterns (`%` and `_` wildcards in user input)
- Check for XSS if any user input is rendered in HTML responses

## 6. Rate Limiting & DoS

- Are HTTP endpoints rate-limited? Without rate limiting, an attacker can:
  - Burn through your Anthropic API credits
  - Overwhelm the VPS with requests
  - Trigger excessive Claude subprocess spawns
- Check that rate limiting uses the real client IP (Cloudflare: `cf-connecting-ip`, otherwise `x-forwarded-for`)

## 7. Resource Management

- Are Claude subprocesses killed on timeout? Zombie processes can accumulate
- Are uploaded/temporary files cleaned up after processing?
- Are there any unbounded in-memory data structures (maps, arrays) that grow without cleanup?

## 8. Logging & Information Disclosure

- Are full message contents logged? Truncate to prevent sensitive data in logs
- Do error responses leak internal details (stack traces, file paths, env var names)?
- Are API error messages sanitized before being sent back to the user?

## Output Format

For each finding, use this format:

### P[0-3]-[number]: [Short title]
**File:** `path/to/file.ts`
**Risk:** [What an attacker could do]
**Fix:** [Specific code change needed]

After listing all findings, help me fix them one at a time, starting with P0.
```

---

## What to Expect

The audit typically finds issues in these areas:

| Priority | Common Findings |
|----------|----------------|
| **P0** | Unrestricted subprocess permissions, unauthenticated endpoints exposing data |
| **P1** | Missing webhook signature verification, path traversal in file uploads, API keys in URLs |
| **P2** | No rate limiting, timing-attack-vulnerable secret comparisons, JWT verification gaps |
| **P3** | Verbose logging, temp file cleanup, hardcoded config values |

## After the Audit

- Fix P0 issues immediately — these allow remote code execution or data exposure
- Fix P1 issues before deploying to production
- P2 and P3 can be addressed incrementally
- Re-run the audit after major changes or new integrations
- Keep your findings documented so you can track what's been fixed
