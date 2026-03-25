---
name: openclaw-scout
description: Daily scout of Reddit and other communities for OpenClaw tips, tricks, new skills, config optimizations, and automation patterns. Scans subreddits, filters for actionable items, and delivers a concise report. Use when you want to stay current on the OpenClaw ecosystem, or set up as a daily cron job.
---

# OpenClaw Scout

Scan Reddit and AI agent communities for OpenClaw improvements, then deliver a concise actionable report.

## When to Use

- As a daily cron job (recommended: 8PM local time)
- On demand when user asks to check for OpenClaw news/tips
- After an OpenClaw update to see what the community is doing with new features

## Workflow

### Step 1: Scan Sources

Search these subreddits for posts from the last 24-48h:

```
site:reddit.com/r/openclaw
site:reddit.com/r/openclawsetup
site:reddit.com/r/openclawusecases
site:reddit.com openclaw agent automation tips
site:reddit.com AI agent cron workflow self-hosted
```

Also check r/LocalLLaMA, r/selfhosted, r/ChatGPTCoding for relevant agent patterns.

Use `web_search` for discovery, then `web_fetch` on interesting posts to get full content + top comments.

### Step 2: Filter for Signal

**Include:**
- New skills or integrations people built
- Config optimizations (compaction, pruning, caching, model routing)
- Creative cron job / automation patterns
- Multi-agent architectures
- Cost-saving tricks (model selection, local models, caching)
- Tools or CLIs that complement OpenClaw
- Workflow patterns transferable to the user's setup

**Exclude:**
- Basic setup questions
- Bug reports without workarounds
- Posts with no actionable takeaway
- Anything the user's setup already does (check cron jobs and config)

### Step 3: Compose Report

Format as a single message:

```
🔭 OpenClaw Scout Report — [date]

Scanned r/openclaw, r/openclawsetup, r/LocalLLaMA + related. Here's what stood out:

**1. [Title]**
[What it is, one line]
[Why it matters for your setup, one line]
⚡ Effort: [quick/medium/big] ([brief justification])
🔗 [source link]

**2. [Title]**
...

📊 Overall: [one-line assessment of the day's findings]
```

Target 3-5 items. Prioritize by impact. Be specific about implementation, not vague.

### Step 4: Log

Append a brief entry to `memory/reddit-scout-log.md` (create if needed):

```
## [YYYY-MM-DD]
- [item 1 title]: [one-line summary]
- [item 2 title]: [one-line summary]
- Nothing notable (if quiet day)
```

This prevents recommending the same thing twice.

## Cron Setup

To run daily at 8PM:

```json
{
  "name": "reddit-openclaw-scout",
  "schedule": { "kind": "cron", "expr": "0 20 * * *", "tz": "USERS_TIMEZONE" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "Run the OpenClaw Scout skill. Scan Reddit for OpenClaw tips and deliver the report to the user.",
    "model": "anthropic/claude-sonnet-4-6",
    "timeoutSeconds": 180
  },
  "delivery": { "mode": "announce" }
}
```

Adjust:
- `tz` to the user's timezone
- `model` to whatever's available (Sonnet works well, Haiku is too shallow for reading Reddit threads)
- Delivery mode: `announce` sends the report to chat, `none` just logs it

## Notes

- The skill uses `web_search` and `web_fetch`, both available by default in OpenClaw
- No API keys needed (Reddit public pages)
- Reddit `.json` trick: append `.json` to any Reddit URL for structured data (avoids HTML parsing)
- If the user wants Telegram/Discord delivery from the cron, set the delivery channel accordingly
- Model recommendation: Sonnet for quality, Haiku won't extract enough signal from long threads
