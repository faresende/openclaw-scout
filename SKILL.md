---
name: openclaw-scout
description: Daily scout of Reddit, Hacker News, GitHub, and Discord for OpenClaw tips, skills, config optimizations, and automation patterns. Scans multiple sources, deduplicates against your setup and past reports, and delivers a concise actionable report. Use on demand or as a daily cron job.
---

# OpenClaw Scout

Scan communities for OpenClaw improvements. Filter for signal. Deliver actionable reports. Learn from past runs.

## Sources (in priority order)

1. **Reddit** — r/openclaw, r/openclawsetup, r/openclawusecases, r/LocalLLaMA, r/selfhosted
2. **Hacker News** — AI agent, automation, and self-hosted topics
3. **GitHub** — Trending repos tagged `openclaw`, `ai-agent`, `llm-tools`
4. **Discord** — OpenClaw community (https://discord.com/invite/clawd) #showcase and #tips channels

## Workflow

### Step 1: Gather

**Reddit** (use `.json` endpoints for structured data, no auth needed):
```bash
# Fetch recent posts as JSON — no API key required
curl -s "https://www.reddit.com/r/openclaw/new.json?limit=25" -H "User-Agent: openclaw-scout/1.0"
curl -s "https://www.reddit.com/r/openclawsetup/new.json?limit=15" -H "User-Agent: openclaw-scout/1.0"
```
Parse `data.children[].data` for `title`, `selftext`, `url`, `created_utc`, `score`, `num_comments`.
Only process posts from the last 48 hours. Skip posts with score < 3 (noise).

**Hacker News** (API, no auth):
```bash
# Search for recent OpenClaw / AI agent posts
curl -s "https://hn.algolia.com/api/v1/search_by_date?query=openclaw&tags=story&numericFilters=created_at_i>$(date -v-2d +%s)"
curl -s "https://hn.algolia.com/api/v1/search_by_date?query=ai+agent+automation&tags=story&numericFilters=created_at_i>$(date -v-1d +%s)"
```

**GitHub Trending** (web scrape, no auth):
```
web_fetch https://github.com/trending?since=daily&spoken_language_code=en
```
Look for repos mentioning openclaw, agent framework, llm tools, mcp.

**Discord** (if browser tool available):
Check #showcase and #tips in the OpenClaw Discord for pinned or high-reaction posts.

**Fallback:** If `.json` endpoints fail (rate limited), fall back to `web_search` queries:
```
site:reddit.com/r/openclaw (last 24h)
site:news.ycombinator.com openclaw OR "ai agent" automation
```

### Step 2: Read and Evaluate

For posts with high signal (score > 10, or detailed writeup, or links to tools/repos):
- `web_fetch` the full post + top comments
- For linked GitHub repos: read the README
- For linked tools: check if actively maintained (last commit < 30 days)

**Score each item 1-5:**
- Impact on daily workflow (1-5)
- Implementation effort (invert: 5=quick, 1=big project)
- Novelty vs what the user already has (5=totally new, 1=already doing it)

Only report items scoring >= 8 total.

### Step 3: Deduplicate

Before including an item, check two things:

**A) Already in our setup?**
Read the user's cron jobs (`cron list`) and skim their config. If we already do something equivalent, skip it or note "you already have this, but their approach to X is different."

**B) Already reported?**
Check `memory/reddit-scout-log.md` for past entries. Don't recommend the same tool/pattern twice unless there's a meaningful update (new version, new approach).

### Step 4: Compose Report

```
🔭 OpenClaw Scout Report — [Day, Mon DD]

Scanned: [sources checked, with post counts]

**1. [Title]** ⭐ [score/15]
[What it is — one line]
[Why it matters for your setup — one line, specific]
⚡ Effort: [quick/medium/big]
🔗 [source URL]

...

📊 [one-line assessment: "Strong day" / "Quiet, nothing actionable" / etc.]
```

Rules:
- 3-5 items max. Quality over quantity.
- If nothing scores >= 8, say so honestly. Don't pad the report.
- Be specific about implementation: "add this to your SOUL.md" not "consider adopting this pattern"
- Include the actual command/config snippet when possible

### Step 5: Log

Append to `memory/reddit-scout-log.md`:

```markdown
## YYYY-MM-DD ([Day])
**Sources:** [list with post counts, e.g. "r/openclaw (12), HN (3), GH trending (1)"]
**Items reported:** [count]
1. [Title] — [one-line summary] — [status: reported/skipped-already-have/skipped-low-signal]
**Quiet?** [yes/no]
```

### Step 6: Track Actions (optional)

If the user acts on an item (installs a skill, changes config, etc.), update the log entry:
```
1. predicate-claw — zero-trust tool auth — ✅ installed 2026-03-28
```

This prevents re-recommending and builds a history of what worked.

## Cron Setup

```json
{
  "name": "openclaw-scout",
  "schedule": { "kind": "cron", "expr": "0 20 * * *", "tz": "USER_TIMEZONE" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "Run the openclaw-scout skill. Check all sources, deduplicate against setup and past reports, deliver the report.",
    "model": "anthropic/claude-sonnet-4-6",
    "timeoutSeconds": 180
  },
  "delivery": { "mode": "announce" }
}
```

**Model guidance:**
- Sonnet or equivalent: good balance of reading comprehension and cost
- Haiku: too shallow, misses signal in long threads
- Opus: overkill for this task

**Timezone:** Set to your local timezone so reports arrive at a reasonable hour.

## Customization

**Change sources:** Edit Step 1 to add/remove subreddits or add other platforms.
**Change threshold:** Lower the score cutoff from 8 to 6 for more items, raise to 10 for only bangers.
**Change frequency:** Run weekly instead of daily if your setup is stable and you don't want noise.
