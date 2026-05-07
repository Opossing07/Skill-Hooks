# Hook Intel
### A Claude Code Skill by [@tenfoldmarc](https://www.instagram.com/tenfoldmarc)

Build a living database of proven viral hooks from your competitors. It scrapes their Instagram every week, finds the posts that went 5x+ their normal views, transcribes the hooks, templatizes them, and gives you a searchable library of frameworks you can steal for your own content. Works in any niche.

---

## What It Does

1. **You give it competitor handles** — 5 to 15 Instagram accounts in your niche
2. **It scrapes their content** and finds outliers (posts that went 5x+ their median views)
3. **Downloads each viral reel**, transcribes the spoken hook, screenshots the on-screen text, and grabs the caption
4. **Templatizes everything** — turns each hook into a reusable `[BRACKET]` framework
5. **Saves it all to a database** (Supabase or a local JSON file — your choice)
6. **Refreshes automatically every week** — only grabs new outliers, never deletes old ones
7. **Lets you browse and filter** — sort by views, outlier ratio, hook type, or find unused templates

---

## Requirements

- A Mac, Linux, or Windows computer
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and working
- An [Apify](https://apify.com) account (free tier works) for Instagram scraping
- `yt-dlp` for downloading reels
- `whisper` for transcription
- `ffmpeg` for screenshots
- Optional: [Supabase](https://supabase.com) project for cloud storage (free tier works — otherwise uses a local file)

Don't worry about connecting these manually — the skill walks you through everything on first run.

---

## Install

### Step 1 — Open your terminal

**Mac:** Press `Command + Space`, type **Terminal**, hit Enter.
**Windows:** Press `Win + R`, type **cmd**, hit Enter. (If you have Git Bash, use that instead.)
**Linux:** Open your terminal app.

### Step 2 — Run this command

Copy-paste this entire line and hit Enter:

```bash
git clone https://github.com/tenfoldmarc/hook-intel-skill ~/.claude/skills/hook-intel
```

Wait for it to finish.

### Step 3 — Open Claude Code

```bash
claude
```

### Step 4 — Run the skill

```
/hook-intel
```

First run walks you through setup — your niche, competitor handles, tool checks, database creation, and initial scrape. Takes about 5 minutes.

---

## Commands

| Command | What it does |
|---------|-------------|
| `/hook-intel` | First-time setup or show status |
| `/hook-intel refresh` | Scrape competitors for new outliers (last 7 days) |
| `/hook-intel top` | Browse all templates ranked by views |
| `/hook-intel top unused` | Only templates you haven't used yet |
| `/hook-intel top carousel` | Only templates that work as carousel first slides |
| `/hook-intel add @handle` | Start tracking a new competitor |
| `/hook-intel remove @handle` | Stop tracking (keeps their hooks in your database) |

---

## What You Get

For every viral outlier, the database stores:

- **Spoken hook** — what they said in the first 3 seconds + template
- **On-screen text hook** — what text was displayed + template
- **Caption hook** — what they wrote in the caption + template
- **Hook type** — Tool Discovery, Replace + Kill Claim, Viewer Callout, etc.
- **Why it works** — psychological analysis
- **Outlier ratio** — how viral it was vs. the account's median
- **Carousel-friendly flag** — whether the template works as a carousel first slide
- **Link to original post** — so you can see the actual video

---

## Pair It With /spy

[`/spy`](https://github.com/tenfoldmarc/spy-skill) is the quick version — instant one-off analysis, no database. `/hook-intel` is the full system. Use `/spy` for quick research, `/hook-intel` for building your permanent hook library.

---

## Updating

```bash
cd ~/.claude/skills/hook-intel && git pull
```

---

## Built By

[@tenfoldmarc](https://www.instagram.com/tenfoldmarc) — Follow for daily AI automation walkthroughs. Real systems, not theory.
