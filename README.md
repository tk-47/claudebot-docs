# Claudebot Setup Guides

Setup guides for [Claudebot](https://github.com/tk-47/go-telegram-bot) — an always-on AI Telegram agent built by GodaGo with [Claude Code](https://claude.ai/claude-code).

Each guide is designed to be **dropped into a Claude Code session** — Claude will walk you through the setup interactively.

---

## Guides

### [Microsoft 365 Integration](ms365-setup.md)
Connect your bot to Outlook Calendar, Email, and Microsoft To Do via the Microsoft Graph API. Ask your bot "What's on my schedule today?" or "Do I have any unread emails?" and get real answers from your actual account.

- Calendar reads/writes with natural language dates
- Unread email summaries
- Microsoft To Do task lists
- Auto-inclusion in daily morning briefings
- ~10 minute setup

### [Tempest Weather Station](tempest-weather-setup.md)
Connect your bot to a WeatherFlow Tempest personal weather station. Get live conditions from your own backyard instead of generic internet forecasts.

- Live conditions: temperature, humidity, wind, pressure, rain, UV, lightning
- 5-day forecast
- Natural language triggers — "Is it cold outside?"
- Intentional Claude bypass for accurate local readings
- ~5 minute setup

### [Google Calendar & Sheets](google-api-setup.md)
Connect your bot to Google Calendar and Google Sheets using lightweight Python CLI scripts and a TypeScript client. Bypasses the broken workspace-mcp server with direct API calls that actually work.

- Calendar: list, create, delete, search events
- Sheets: read, write, append, create spreadsheets
- Drive: search spreadsheets by name
- Shared OAuth credentials — one auth flow covers everything
- ~15 minute setup

### [Microsoft Teams Integration](teams-setup.md)
Add Teams as a second messaging platform with synchronized conversations. Messages from Telegram and Teams share the same history — Claude sees full context regardless of where you sent from.

- Synchronized cross-platform conversations
- Human-in-the-loop via Adaptive Cards
- Same hybrid routing (VPS + local Mac)
- No botbuilder SDK — direct Bot Framework REST API
- ~30 minute setup

### [Fireflies.ai Meeting Transcripts](fireflies-setup.md)
Connect your bot to Fireflies.ai so meeting transcripts are automatically stored in memory. Summaries become searchable facts, action items become trackable goals, and you get a Telegram notification when it's done.

- Automatic meeting summary storage
- Action items saved as trackable goals
- Webhook-based — no polling, no manual steps
- Searchable via semantic memory — "What did we discuss in the Q1 meeting?"
- ~10 minute setup

### [Supabase to Convex Migration](supabase-to-convex-migration.md)
Migrate your bot's database from Supabase to Convex with zero downtime. Uses a backend adapter pattern so you can run both databases in parallel and roll back instantly.

- Schema translation: SQL → TypeScript, pgvector → Convex vector search
- Drop-in client adapter with identical function signatures
- Data migration script that preserves existing embeddings
- Dual-write verification phase before cutover
- ~1 day active work + 2 days monitoring

---

## How to Use

1. Copy the `.md` file for the integration you want
2. Drop it into your Claudebot project root (or paste it into a Claude Code session)
3. Tell Claude: **"Set up [integration name]"**
4. Follow the interactive prompts

---

## About Claudebot

Claudebot is an always-on Telegram agent that relays your messages to Claude and sends back responses. It supports hybrid deployment (local Mac + VPS), persistent memory, proactive check-ins, morning briefings, and direct API integrations for calendar, email, weather, and more.

The main repo is private, but these guides are published here for the community.
