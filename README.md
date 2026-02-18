# Claudebot Setup Guides

Setup guides for [Claudebot](https://github.com/tk-47/go-telegram-bot) — an always-on AI Telegram agent built with [Claude Code](https://claude.ai/claude-code).

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
