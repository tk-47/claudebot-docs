# Oura Ring Health Integration

> **For Claude Code:** Drop this file into your project root or paste it into a Claude Code session. Claude will walk you through each step interactively. When you're ready, say: **"Set up Oura Ring integration"**

Connect your Telegram bot to an [Oura Ring](https://ouraring.com/) so your AI assistant understands your body. Once set up, the bot can:

- Report sleep scores, readiness, activity, and stress levels
- Show detailed sleep breakdowns — stages, HRV, heart rate, breathing rate
- Include health data in daily morning briefings
- Factor sleep quality and readiness into proactive smart check-ins
- Answer on-demand questions — "How did I sleep?", "What's my readiness score?"

---

## How It Works

The bot calls the [Oura API v2](https://cloud.ouraring.com/v2/docs) directly via HTTP using a Personal Access Token. No OAuth flow, no MCP server, no web search — just direct REST calls to your ring's cloud data.

Health data flows into the bot through three paths:

```
Morning Briefing (scheduled, direct API)
  │
  └── getOuraSummary() — src/lib/oura.ts
        └── GET api.ouraring.com/v2/usercollection/daily_sleep
        └── GET api.ouraring.com/v2/usercollection/daily_readiness
        └── GET api.ouraring.com/v2/usercollection/daily_activity
        └── GET api.ouraring.com/v2/usercollection/daily_stress
        └── Formatted summary appears in briefing:
              "💍 HEALTH (Oura)
               Sleep: 76/100
                 ⚠ Low: deep sleep 21, efficiency 58
               Readiness: 81/100
                 Body temp: +0.2°C from baseline
               Activity: 2,579 steps | 169 active cal"

Smart Check-in (scheduled, context injection)
  │
  └── getOuraSummary() injected into Claude's decision prompt
        └── Claude factors health into tone and recommendations
        └── e.g., "You didn't sleep great — maybe take it easy today"

On-Demand Query (user asks via Telegram)
  │
  └── Claude subprocess runs: bun run src/tools/oura-cli.ts sleep
        └── Returns detailed sleep report with stages, HR, HRV
        └── Claude formats and replies
```

---

## Prerequisites

- An [Oura Ring](https://ouraring.com/) (Gen 2 or Gen 3) synced to the Oura app
- An Oura account (the one linked to your ring)
- [Bun](https://bun.sh/) runtime installed

---

## Step 1: Create a Personal Access Token

### What you need to do:

1. Go to [cloud.ouraring.com](https://cloud.ouraring.com/) and sign in with your Oura account
2. Navigate to **Personal Access Tokens** ([cloud.ouraring.com/personal-access-tokens](https://cloud.ouraring.com/personal-access-tokens))
3. Click **Create New Personal Access Token**
4. Give it a name like "Claudebot"
5. Copy the token — it's a long alphanumeric string like `ABCDEFGHIJKLMNOPQRSTUVWXYZ123456`

The token gives read access to all your ring data. No scopes to configure.

### Tell Claude Code:
"Here's my Oura access token: [TOKEN]"

---

## Step 2: Add to Environment

### What Claude Code does:
- Adds `OURA_ACCESS_TOKEN` to your `.env` file

```env
# Oura Ring
OURA_ACCESS_TOKEN=your-personal-access-token
```

### Tell Claude Code:
"Add the Oura token to .env"

---

## Step 3: Verify the Connection

### What Claude Code does:
- Runs the Oura CLI to test connectivity
- Confirms `isOuraEnabled()` returns true

```bash
bun run src/tools/oura-cli.ts summary
```

Expected output (scores vary based on your data):
```
Sleep: 76/100 (2025-02-16)
  ⚠ Low: deep sleep 21, efficiency 58
Readiness: 81/100
  Body temp: +0.2°C from baseline
Activity: 2,579 steps | 169 active cal
```

If you see "No Oura data available — ring may not have synced yet", open the Oura app on your phone and let it sync, then try again.

### Tell Claude Code:
"Test the Oura Ring integration"

---

## What Data Is Available

### Daily Summary (via `oura-cli.ts summary`)

| Field | Example |
|-------|---------|
| Sleep score | 76/100 |
| Low sleep contributors | deep sleep 21, efficiency 58 |
| Readiness score | 81/100 |
| Body temperature deviation | +0.2°C from baseline |
| Low readiness contributors | sleep balance 70 |
| Activity (steps + calories) | 2,579 steps \| 169 active cal |
| Stress summary | restored, normal, or stressful |

### Detailed Sleep (via `oura-cli.ts sleep`)

| Field | Example |
|-------|---------|
| Sleep score | 76/100 |
| Bedtime | 10:41 PM → 9:27 AM |
| Total sleep | 7h 56m |
| Deep sleep | 14m |
| REM sleep | 1h 39m |
| Light sleep | 6h 4m |
| Awake time | 2h 49m |
| Efficiency | 74% |
| Sleep latency | 9m |
| Avg heart rate | 60 bpm |
| Lowest heart rate | 55 bpm |
| Avg HRV | 29 ms |
| Avg breathing rate | 16 breaths/min |
| Contributors breakdown | deep sleep 21, efficiency 58, latency 83, rem sleep 95, restfulness 60, timing 77, total sleep 96 |

### Readiness (via `oura-cli.ts readiness`)

| Field | Example |
|-------|---------|
| Readiness score | 81/100 |
| Body temp deviation | +0.19°C |
| Body temperature | 89/100 |
| HRV balance | 82/100 |
| Previous night | 75/100 |
| Recovery index | 100/100 |
| Resting heart rate | 78/100 |
| Sleep balance | 70/100 |
| Sleep regularity | 83/100 |

### Activity (via `oura-cli.ts activity`)

| Field | Example |
|-------|---------|
| Activity score | (available when day completes) |
| Steps | 2,579 |
| Active calories | 169 / 450 target |
| Total calories | 2,384 |
| Walking distance | 2.5 km |
| High activity | 0m |
| Medium activity | 7m |
| Low activity | 2h 57m |
| Sedentary time | 14h 48m |
| Distance to target | 5.3 km |

---

## Where Oura Data Appears

| Context | What's Included |
|---------|----------------|
| **Morning briefing** | Sleep score, readiness score, activity summary, stress level — appears between weather and Orthodox calendar |
| **Smart check-in** | Full summary injected into Claude's decision context — Claude adapts tone based on sleep quality and readiness |
| **On-demand queries** | Claude subprocess calls `oura-cli.ts` for detailed reports when you ask "How did I sleep?" or "What's my readiness?" |

---

## CLI Reference

The Claude subprocess (and you manually) can query Oura data via:

```bash
# Today's scores (sleep, readiness, activity, stress)
bun run src/tools/oura-cli.ts summary

# Detailed sleep report (most recent, or specific date)
bun run src/tools/oura-cli.ts sleep
bun run src/tools/oura-cli.ts sleep 2025-02-14

# Readiness breakdown with all contributors
bun run src/tools/oura-cli.ts readiness
bun run src/tools/oura-cli.ts readiness 2025-02-14

# Activity breakdown with steps, calories, time splits
bun run src/tools/oura-cli.ts activity
bun run src/tools/oura-cli.ts activity 2025-02-14
```

All commands use a 3-day lookback window by default, so if your ring hasn't synced today, it returns the most recent data available.

---

## Customization

### Lookback Window

By default, the bot looks back 3 days to find the most recent data (Oura data can lag if the ring hasn't synced). To change this, edit the default in `src/lib/oura.ts`:

```typescript
// In getOuraSummary():
const { start, end } = getDateRange(3);  // Change 3 to desired days
```

### Timezone

Sleep times and date calculations use `USER_TIMEZONE` from your `.env`. Make sure it's set:

```env
USER_TIMEZONE=America/Chicago
```

---

## VPS Setup

No extra setup needed — just add the same env var to your VPS `.env`:

```env
OURA_ACCESS_TOKEN=your-personal-access-token
```

The Oura API is a public REST endpoint authenticated by Bearer token. No OAuth refresh, no expiration (personal access tokens don't expire unless revoked). Works identically on local and VPS.

---

## Architecture

### New Files

| File | Purpose |
|------|---------|
| `src/lib/oura.ts` | Direct REST client for Oura API v2 — `isOuraEnabled()`, `getOuraSummary()`, `getDetailedSleep()`, `getReadinessDetail()`, `getActivityDetail()` |
| `src/tools/oura-cli.ts` | CLI wrapper so the Claude subprocess can query health data on demand |

### Modified Files

| File | Changes |
|------|---------|
| `.env` / `.env.example` | Added `OURA_ACCESS_TOKEN` |
| `src/morning-briefing.ts` | Oura health section between weather and Orthodox calendar |
| `src/smart-checkin.ts` | Oura data injected into Claude's decision context |
| `src/agents/base.ts` | `BASE_CONTEXT` updated with Oura CLI tool instructions |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "OURA_ACCESS_TOKEN must be set" | Check that the env var is in your `.env` file |
| "Oura API error: 401" | Token is invalid or revoked — create a new one at cloud.ouraring.com/personal-access-tokens |
| "No Oura data available — ring may not have synced yet" | Open the Oura app on your phone and let it sync. Data can lag hours behind |
| Sleep data is from 2-3 days ago | The ring needs to sync via Bluetooth to the Oura app, which then uploads to the cloud. The 3-day lookback handles this gracefully |
| Activity score shows null | Activity scores are only calculated after the day completes — check the next day |
| Stress shows null | Stress tracking requires Oura Gen 3 and may not be available on all plans |
| Wrong sleep times displayed | Check `USER_TIMEZONE` in `.env` — sleep times are converted from UTC using this |
| Bot doesn't answer "How did I sleep?" | The Claude subprocess needs the Oura CLI in `BASE_CONTEXT`. Verify `src/agents/base.ts` includes the `oura-cli.ts` tool instructions |

---

## API Reference

The bot uses the [Oura API v2](https://cloud.ouraring.com/v2/docs) with these endpoints:

```
GET https://api.ouraring.com/v2/usercollection/daily_sleep
GET https://api.ouraring.com/v2/usercollection/daily_readiness
GET https://api.ouraring.com/v2/usercollection/daily_activity
GET https://api.ouraring.com/v2/usercollection/daily_stress
GET https://api.ouraring.com/v2/usercollection/sleep

Headers:
  Authorization: Bearer YOUR_PERSONAL_ACCESS_TOKEN

Query params:
  start_date=YYYY-MM-DD
  end_date=YYYY-MM-DD
```

All endpoints return `{ data: [...], next_token: null }`. The bot fetches 3 days of data and takes the most recent entry. No documented rate limits for personal access tokens.

---

## References

- [Oura Ring](https://ouraring.com/) — The hardware
- [Oura API v2 Documentation](https://cloud.ouraring.com/v2/docs) — REST API docs
- [Oura Developer Portal](https://cloud.ouraring.com/) — Token management and app registration
- [@pinta365/oura-api](https://jsr.io/@pinta365/oura-api) — TypeScript library reference (not used directly, but useful for type definitions)
