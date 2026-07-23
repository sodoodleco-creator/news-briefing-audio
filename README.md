# Morning Intelligence Brief — Investment Committee Style

A formal "research desk memo" styled daily briefing: Spectral serif
headlines, parchment tones, a classification bar, roman-numeral sections
(Executive Summary, Top Developments, Thematic Watch, Markets & Macro,
Technology & AI, Closing Note), and a real live market data table — sent
every morning at 6:30am IST. Curated with India macro/micro trends as an
explicit top-priority category alongside global business, AI, and
markets news.

**Cost: $0.** Curation runs on Gemini's free API tier, market data comes
from Yahoo Finance (free, no key), and everything else (GitHub Actions,
Gmail sending) is free-forever infrastructure. If Gemini's call fails,
the script falls back to a simpler local summarizer for that run — the
email still arrives, just with only the Top Developments section (no
executive summary, thematic watch, AI pulse, or closing note, since those
need AI synthesis), and the footer honestly says which mode produced it.
If the market data fetch fails for any reason, Markets & Macro is simply
omitted that day rather than showing stale or fabricated numbers.

"Recommended Reading" from the original design reference isn't included,
since it would need a long-form-essay picker this pipeline doesn't have.
The Closing Note is an original insight rather than a quote attributed to
a real person, to avoid ever misattributing something to someone who
didn't say it.

## How it works
1. `config.yaml` lists RSS feeds — your global publishers (Reuters, AP,
   BBC, TechCrunch, Ars Technica, MIT Tech Review, The Verge, OpenAI,
   Anthropic, DeepMind, etc.) plus India-focused sources (Economic Times
   Markets, LiveMint, Moneycontrol, Business Standard, RBI) — and
   `recipient_name` for the "Good morning, ___." greeting.
2. Every day, GitHub's servers run `news_briefing.py`, which:
   - fetches recent articles from each feed
   - merges duplicate stories multiple sources are covering
   - sends the candidates to Gemini along with the curation spec (which
     weighs India macro/micro as a top-priority category), getting back
     an executive summary, 5-7 ranked developments (assessment, key
     points, materiality score each), an optional thematic-watch
     opportunity, a few AI Pulse items, and a closing insight
   - separately pulls live Nifty 50, Sensex, USD/INR, India VIX, Brent
     Crude, and Gold levels via Yahoo Finance (no API key needed)
   - renders everything into the styled HTML email and sends it via Gmail
3. A GitHub Actions cron schedule triggers this at 6:30am IST daily —
   fixed forever since India has no daylight saving time.

## Setup (about 15 minutes)

### 1. Sources and name (already filled in)
`config.yaml` already lists your publishers with RSS URLs pre-filled,
plus `recipient_name` (defaults to "Mohin" — change to whatever name you
want in the greeting). A few publishers (Reuters, AP, Anthropic, Meta
Business Blog, Google Ads Blog, LinkedIn Marketing Blog) don't run a
public RSS feed, so those route through a Google News "site:" search
instead. The India-focused feeds (LiveMint, Moneycontrol, Business
Standard) use commonly-documented URLs that weren't individually
re-verified — if `fetch_articles` logs a `[warn]` for one, search
`"<site> RSS feed"` to find the current URL. Same trick works for adding
any new source: `"<site name>" RSS feed`, or try `/feed`, `/rss`,
`/rss.xml`.

### 2. Create a Gmail "app password"
Don't use your real Gmail password here. Instead:
1. Google Account → Security → 2-Step Verification (must be turned on).
2. Go to https://myaccount.google.com/apppasswords
3. Create an app password named e.g. "news-briefing" — copy the
   16-character code.

### 3. Get a free Gemini API key
1. Go to https://aistudio.google.com/app/apikey and sign in.
2. Click **Create API key** (no credit card needed for the free tier).
3. Copy the key.
4. This uses `gemini-2.5-flash`, on Google's free tier as of mid-2026.
   Free-tier model names/limits do occasionally change — if the script's
   fallback keeps kicking in, check
   https://ai.google.dev/gemini-api/docs/pricing for the current free
   Flash model name and update the `GEMINI_MODEL` line near the top of
   `news_briefing.py`.

### 4. Create a GitHub repository
1. https://github.com/new → create a new **private** repository.
2. Upload every file from this folder to the repo root (including the
   hidden `.github/workflows/daily-briefing.yml` — check it actually
   landed there, since drag-and-drop sometimes misses hidden folders or
   nests everything one level too deep).

### 5. Add your secrets
Repo → **Settings → Secrets and variables → Actions → New repository
secret**. Add these four:

| Secret name           | Value                                      |
|------------------------|---------------------------------------------|
| `EMAIL_ADDRESS`       | the Gmail address to send FROM              |
| `EMAIL_APP_PASSWORD`  | the 16-character app password from step 2   |
| `TO_EMAIL`            | the address to send the briefing TO (can be the same Gmail address) |
| `GEMINI_API_KEY`      | your Gemini API key from step 3             |

(If you skip `GEMINI_API_KEY`, the script runs the free local summarizer
instead — it degrades gracefully rather than failing.)

### 6. Test it
**Actions** tab → "Daily News Briefing" → **Run workflow**. Check your
inbox within a minute or two. If it fails, click into the run to read the
error log.

### 7. Let it run
Once the test succeeds, it fires automatically every day at 6:30am IST —
no seasonal time-zone adjustment ever needed.

## Customizing
- **Recipient name**: `recipient_name` in `config.yaml`.
- **What's prioritized / India weighting**: edit the `BRIEFING_SPEC` text
  block near the top of `news_briefing.py` — the actual instruction
  Gemini curates against.
- **More/fewer stories**: `BRIEFING_SPEC` targets 5-7; adjust the wording
  there, and `max_stories` in `fallback_curate()` to match.
- **Market tickers**: edit the `tickers` list in `fetch_market_data()` —
  any valid Yahoo Finance symbol works (e.g. `^GSPC` for the S&P 500, or
  `RELIANCE.NS` for a specific Indian stock).
- **Colors/fonts**: constants (`ACCENT`, `INK`, `FONT_SERIF`, `FONT_MONO`,
  etc.) near the top of the rendering section in `news_briefing.py`.
- **Send time**: the `cron` line in the workflow file (UTC).
- **Non-Gmail email**: `smtp.gmail.com` / port in `send_email()`.
