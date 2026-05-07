---
name: hook-intel
description: "Full hook intelligence system — scrape competitors, find viral outliers, build a living database of proven hooks and templates that gets smarter every week."
---

# /hook-intel

Full hook intelligence system — scrape competitors, find viral outliers, build a living database of proven hooks and templates that gets smarter every week. Set it and forget it.

## Usage
- `/hook-intel` — first-time setup OR show status if already configured
- `/hook-intel refresh` — manually trigger a refresh (scrape for new outliers)
- `/hook-intel top` — browse top templates and hooks
- `/hook-intel top unused` — only templates you haven't used yet
- `/hook-intel top carousel` — only carousel-friendly templates
- `/hook-intel add @handle` — add a new competitor to track
- `/hook-intel remove @handle` — stop tracking a competitor

---

## Step 0 — First-Time Setup

Check if config exists at `~/.claude/skills/hook-intel/config.json`.

**If config exists:** Load config, show status summary, proceed to requested command.

**If config does NOT exist:** Run the setup wizard.

```
Welcome to /hook-intel! Let's get you set up. This takes about 2 minutes and only happens once.
```

### Ask Questions

1. **"What's your niche?"** — e.g. "AI automation", "fitness coaching", "ecommerce"
2. **"What's your Instagram handle?"** (optional — for tracking which templates you've used)
3. **"Give me 5-15 Instagram handles of competitors in your niche"** — the more the better
4. **"Do you have a Supabase project you want to store data in?"**
   - If YES: ask for project URL and anon key
   - If NO: "No problem — I'll use a local JSON file instead. You can connect Supabase later."

### Check Apify MCP
Try calling `mcp__apify__search-actors` with query "instagram".
- If it works: "Apify connected."
- If it fails:
  ```
  Apify is not connected. To set it up:
  1. Create a free account at https://apify.com
  2. Go to Settings → Integrations and copy your API token
  3. Exit Claude Code (type /exit)
  4. Run: claude mcp add apify -- npx -y @anthropic-ai/apify-mcp-server
  5. When prompted, paste your API token
  6. Reopen Claude Code and run /hook-intel again
  ```
  Stop here until connected.

### Check CLI Tools

```bash
YT_DLP=$(which yt-dlp 2>/dev/null)
WHISPER=$(which whisper 2>/dev/null)
FFMPEG=$(which ffmpeg 2>/dev/null)
```

For each missing tool, give install instructions and stop until all are present.

### Check Supabase MCP (if using Supabase)
If the user wants Supabase storage, verify `mcp__supabase__execute_sql` is available.
- If not connected, give setup instructions.

### Create Database

**If Supabase:** Create tables via `mcp__supabase__apply_migration`:

```sql
CREATE TABLE IF NOT EXISTS hook_intel_posts (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  post_url text UNIQUE NOT NULL,
  handle text NOT NULL,
  is_own boolean DEFAULT false,
  spoken_hook text,
  on_screen_text text,
  caption_hook text,
  full_transcript text,
  spoken_template text,
  screen_template text,
  caption_template text,
  hook_type text,
  why_it_works text,
  content_angle text,
  views integer DEFAULT 0,
  likes integer DEFAULT 0,
  shares integer DEFAULT 0,
  outlier_ratio real,
  is_favorite boolean DEFAULT false,
  posted_at timestamptz,
  created_at timestamptz DEFAULT now()
);

CREATE TABLE IF NOT EXISTS hook_intel_templates (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  spoken_template text,
  screen_template text,
  caption_template text,
  example_spoken text,
  example_screen text,
  example_caption text,
  example_handle text,
  views integer DEFAULT 0,
  outlier_ratio real,
  hook_type text,
  source_post_url text,
  carousel_friendly boolean DEFAULT false,
  created_at timestamptz DEFAULT now()
);
```

**If local:** Create `~/.claude/skills/hook-intel/data/hooks.json`:
```json
{ "posts": [], "templates": [] }
```

### Save Config

Save to `~/.claude/skills/hook-intel/config.json`:
```json
{
  "niche": "AI automation",
  "ownHandle": "@theirhandle",
  "competitors": ["handle1", "handle2"],
  "storage": "supabase" | "local",
  "supabase": {
    "projectId": "xxx",
    "anonKey": "xxx",
    "url": "https://xxx.supabase.co"
  },
  "toolPaths": {
    "ytDlp": "/path/to/yt-dlp",
    "whisper": "/path/to/whisper",
    "ffmpeg": "/path/to/ffmpeg"
  },
  "setupComplete": true,
  "setupDate": "YYYY-MM-DD",
  "lastRefresh": null,
  "cronEnabled": false
}
```

### Run Initial Scrape

After setup, automatically run the first scrape (same as `/hook-intel refresh` but for the last 30 days instead of 7).

### Show Summary

```
Hook Intel is set up!

📊 [N] competitors tracked
🔥 [N] outliers found and processed
🧩 [N] templates generated

Commands:
  /hook-intel refresh  — scan for new outliers
  /hook-intel top      — browse your best templates
  /hook-intel add @x   — add a competitor
  /spy @x @y           — quick one-off analysis

Want me to set up automatic weekly refresh? (requires perma-cron skill)
```

---

## /hook-intel refresh

1. Read config from `~/.claude/skills/hook-intel/config.json`
2. Scrape all competitors via Apify (50 posts each for median calculation)
3. Filter to **last 7 days only** for outlier candidates
4. Skip posts already in the database (check post_url)
5. Find **5x+ outliers** from the new posts
6. For each new outlier: download, transcribe, screenshot, extract all three hooks (spoken, on-screen, caption), templatize, analyze
7. Save to database — **APPEND ONLY, never delete or update existing entries**
8. Update `lastRefresh` in config
9. Print summary of what was found

---

## /hook-intel top

1. Read all hooks from the database
2. Group by template pattern
3. Rank by total views across all posts using that template

### Filters:
- **`unused`** — compare templates against user's own posts (if ownHandle is set). Only show templates they haven't used yet.
- **`carousel`** — only show carousel-friendly templates:
  - YES: Replace + Kill Claim, Listicle, Framework, Tool Discovery, Viewer Callout, Comparison, Don't Say / Instead Say
  - NO: POV/Meme, Comment Reply, Speed Tutorial Rapid Steps, Contrarian rants, Celebrity/News

### Display for each template:
```
1. [TOOL] just killed [PROFESSION]

   🎙️ Spoken:    [TOOL] just killed [PROFESSION]
   📱 On-screen: [TOOL] JUST KILLED [PROFESSION] [EMOJI]
   📝 Caption:   [TOOL] just killed [PROFESSION]. Comment [KEYWORD] for the guide.

   Best example: @creator — 392K views (128x outlier)
   Used by: @creator1, @creator2, @creator3
   Combined: 1.2M views across 4 posts
   Carousel: ✓

   🔗 https://www.instagram.com/p/xxxxx/
```

---

## /hook-intel add @handle

1. Add the handle to config.json competitors list
2. Run immediate scrape of just that handle (50 posts)
3. Find outliers and process them
4. Save to database
5. Confirm: "Added @handle — found [N] outliers"

---

## /hook-intel remove @handle

1. Remove from config.json competitors list
2. Do NOT delete their hooks from the database (data is sacred)
3. Confirm: "Stopped tracking @handle. Their [N] existing hooks remain in your database."

---

## Important Rules

- **APPEND ONLY** — never delete or update existing hooks or templates in the database
- **Three hooks per post** — always extract spoken, on-screen text, AND caption. All three get their own template.
- **Carousel flag** — every template gets tagged as carousel-friendly or not
- **Memes included but flagged** — POV/Meme posts get saved but noted as "high views, low follow conversion"
- **Works with or without Supabase** — local JSON is the fallback for people without Supabase
- **Own handle is optional** — if provided, enables the "unused" filter in `/hook-intel top`

---

Built by [@tenfoldmarc](https://instagram.com/tenfoldmarc). Follow for daily AI automation builds — real systems, not theory.
