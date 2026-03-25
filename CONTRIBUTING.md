# Contributing

## Install

```bash
openclaw skill install gh:faresende/openclaw-scout
```

## Report a Bug or Suggest an Improvement

Open an issue: https://github.com/faresende/openclaw-scout/issues

Include what happened vs what you expected, and your OpenClaw version if relevant.

## Submit a Change

1. Fork the repo
2. Edit `SKILL.md` (that's the whole skill)
3. Open a pull request with a short description of what you changed and why

Keep `SKILL.md` under 500 lines. No extra files unless it's a script in `scripts/` or a reference doc in `references/`.

## What Makes a Good Contribution

- New sources (a subreddit, a Discord, an RSS feed) with the fetch command
- Better scoring heuristics
- Dedup improvements
- Fixes for broken API endpoints
- Model recommendations from real usage

## Style

- Imperative voice ("Fetch posts" not "You should fetch posts")
- Concrete commands over vague instructions
- If you add a source, include the exact curl/fetch command
