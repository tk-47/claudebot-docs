# Claudebot Security Audit — 2026-02-24

Full security audit of the Claudebot codebase. Fixes are applied incrementally.

## P0-1: Removed `--dangerously-skip-permissions` from bot subprocesses
**Status: FIXED (2026-02-24)**
**File:** `src/lib/claude.ts`

### What was wrong
Both `callClaude()` and `runClaudeWithTimeout()` passed `--dangerously-skip-permissions` to every Claude Code subprocess spawned by the bot. This gave full unrestricted access (Bash, Write, Edit, all tools) to the subprocess, meaning any prompt injection in untrusted context (email subjects, calendar titles, meeting transcripts, document filenames) could execute arbitrary commands on the Mac.

### What was changed
- Replaced `--dangerously-skip-permissions` with `--allowedTools` using a read-only allowlist: `Read, Glob, Grep, WebSearch, WebFetch`
- Callers can still override via the `allowedTools` option in `ClaudeOptions`
- Added `DEFAULT_ALLOWED_TOOLS` constant with documentation

### Why this is safe
The bot already has direct API integrations for all external services:
- `src/lib/ms365.ts` — calendar, email, tasks (Microsoft Graph API)
- `src/lib/sheets.ts` — Google Sheets (direct OAuth)
- `src/lib/weather.ts` — Tempest weather station
- `src/lib/flightaware.ts` — FlightAware AeroAPI
- `src/lib/voice.ts` — ElevenLabs TTS/calls
- `src/lib/transcribe.ts` — Gemini transcription

Claude subprocess is only needed for conversational AI. It doesn't need Bash or Write.

### What to watch for
- If a new feature genuinely needs the bot's Claude subprocess to run commands, add the specific tool to the `allowedTools` array for that call — don't re-add `--dangerously-skip-permissions`
- Interactive Claude Code sessions (terminal) are NOT affected by this change

### Rollback
If the bot breaks due to this change, revert `DEFAULT_ALLOWED_TOOLS` or pass a broader `allowedTools` for the specific call that needs it. As a last resort, re-add `--dangerously-skip-permissions` to the args array in both functions, but understand the risk.

---

## Remaining issues (in priority order)

### P0-2: `/context` endpoint unauthenticated
**Status: FIXED (2026-02-24)** — Endpoint disabled entirely (returns 404). Voice agent is not configured yet. When re-enabling, add Bearer token auth using `GATEWAY_SECRET` before returning any data. See comment in `src/vps-gateway.ts`.

### P0-3: CORS `Access-Control-Allow-Origin: *` on all VPS endpoints
**Status: FIXED (2026-02-24)** — Removed all CORS headers and OPTIONS preflight handler. All VPS callers (Telegram, GitHub, Teams, ElevenLabs) are server-to-server and don't need CORS. If a web UI is added later, add CORS to that specific endpoint with a specific origin — not a wildcard.

### P1-1: ElevenLabs webhook `/webhook/elevenlabs` has no authentication
**Status: FIXED (2026-02-24)** — Endpoint disabled (returns 404). Voice agent not configured. When re-enabling, add shared secret or signature verification before processing payloads.

### P1-2: Fireflies webhook signature verification defaults to open
**Status: FIXED (2026-02-24)** — Flipped default from `return true` to `return false` when `FIREFLIES_WEBHOOK_SECRET` is not set. Unsigned webhooks are now rejected. Set the secret in `.env` to enable Fireflies integration.

### P1-3: File upload path traversal
**Status: FIXED (2026-02-24)** — Filename sanitized with `basename()` + `..` replacement in `src/bot.ts`. A crafted filename like `../../.env` is now reduced to `.env` (basename strips directory components) and any remaining `..` sequences are replaced with `_`. Files always land inside `uploads/`.

### P1-4: Gemini API key in URL query parameter
**Status: FIXED (2026-02-24)** — Moved API key from `?key=` query parameter to `x-goog-api-key` header in both `transcribeAudio()` and `transcribeAudioBuffer()` in `src/lib/transcribe.ts`. Key no longer appears in server logs, proxy logs, or error messages containing the URL.

### P2-1: No rate limiting on VPS HTTP endpoints
**Status: FIXED (2026-02-24)** — Added in-memory per-IP sliding window rate limiter in `src/vps-gateway.ts`. 30 requests/minute per IP. /health exempt. Uses `cf-connecting-ip` header (Cloudflare) or `x-forwarded-for` fallback. Stale entries purged every 5 minutes.
### P2-2: Non-constant-time secret comparison (timing attack)
**Status: FIXED (2026-02-24)** — Replaced all `===`/`!==` secret comparisons with `crypto.timingSafeEqual()` in 4 locations: deploy webhook HMAC (`vps-gateway.ts`), Telegram webhook secret (`vps-gateway.ts`), Fireflies webhook HMAC (`fireflies.ts`), and voice server token (`voice-server.ts`).
### P2-3: JWT issuer bypass in Teams auth fallback
**Status: FIXED (2026-02-24)** — Removed the "last resort" issuer-less JWT verification fallback in `src/lib/teams-auth.ts`. Tokens must now match a known issuer (Bot Framework, emulator, or configured tenant). Without this, any Azure AD tenant could forge tokens with the correct audience.
### P2-4: ilike pattern injection in Supabase queries (unsanitized `%` and `_`)
**Status: REMOVED (2026-02-24)** — `src/lib/supabase.ts` deleted entirely. Convex is now the sole backend. Convex queries don't use SQL LIKE patterns.
### P2-5: Supabase anon key fallback (RLS dependency)
**Status: REMOVED (2026-02-24)** — `src/lib/supabase.ts` deleted entirely. Convex uses a single URL-based auth model.

### P3-1: Console logging of sensitive message content
**Status: FIXED (2026-02-24)** — Truncated voice transcription log to 50 chars (was 80) for consistency. All message logs already truncate to 50 chars. Low risk — logs are server-side only.

### P3-2: Uploaded files never cleaned up
**Status: FIXED (2026-02-24)** — Added `unlink()` after Claude processes photos and documents in `src/bot.ts`. Voice files already had cleanup. Prevents disk space growth from accumulated temp files.

### P3-3: `.env.local` may be in git history
**Status: NOT AN ISSUE** — Verified `.env.local` was never committed to git. `.gitignore` already covers `.env.*`.

### P3-4: Hardcoded attendance spreadsheet ID
**Status: FIXED (2026-02-24)** — Moved from hardcoded string to `ATTENDANCE_SHEET_ID` env var in `src/bot.ts`. Added to `.env.example`.
