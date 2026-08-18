# Vocario

A vocabulary flashcard app with a real dictionary, an Anki-style spaced repetition scheduler, and points/levels/streaks. Static site — no build step, no framework, no server to run. Cards and accounts live in Supabase; the file itself is plain HTML/CSS/JS.

## Deploy to Cloudflare Pages

**Direct upload (fastest, and doesn't burn build quota)**
1. Go to https://dash.cloudflare.com → *Workers & Pages* → *Create* → *Pages* → *Upload assets*.
2. Name the project (e.g. `vocario`).
3. Drag this folder in, click *Deploy site*.
4. You get a live HTTPS URL at `https://<project>.pages.dev`.

To publish an update later, open the project → *Create new deployment* → drag the folder again.

**From Git (better once you're iterating)**
1. Push this folder to a GitHub repo.
2. Cloudflare Pages → *Connect to Git* → pick the repo.
3. Framework preset: **None**. Build command: leave **empty**. Output directory: `/`
4. Every push to the main branch redeploys automatically.

HTTPS is on by default, which the offline mode and installable app both require.

## Look & scoring

Retro 8-bit theme — pixel fonts (Press Start 2P for headings/HUD, VT323 for body text), hard block shadows, chunky borders, no rounded corners.

Points, levels, and streaks are all **derived from data that already syncs** — nothing extra to keep consistent across devices:
- **+10 points** for filing a new word
- **+5 points** for every review, regardless of grade — deliberately flat, so there's never a reason to grade generously just to protect a score (that would corrupt the actual spaced-repetition scheduling)
- **Level up** every 100 points
- **Streak** counts consecutive days with at least one review (currently tracked per-device, not yet synced — a good candidate for a future upgrade once the leaderboard is built)

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app — markup, styles, logic |
| `sw.js` | Service worker; caches the app shell so reviews work offline |
| `manifest.webmanifest` | Makes it installable to a phone home screen |
| `icon.svg` | App icon |
| `_headers` | Security and cache headers (read automatically by Cloudflare Pages) |

## APIs used

- `api.dictionaryapi.dev` — definitions, part of speech, examples, IPA, pronunciation audio. Free, no key, no rate limit published.
- `api.datamuse.com` — synonyms, only when the dictionary returns none. Free, no key.

Both are called straight from the browser. Nothing is proxied, so there's no backend to run.

## Keyboard

| Key | Action |
|---|---|
| `Space` / `Enter` | Reveal card, then answer *Good* |
| `1` `2` `3` `4` | Again / Hard / Good / Easy |
| `U` | Undo last answer |
| `S` | Hear the word |
| `1`–`4` outside study | Switch drawers |

## Scheduling

SM-2 with Anki-style learning steps.

- New cards: 1 min → 10 min → graduate at 1 day (4 days if answered *Easy*).
- Reviews: interval × ease, ease starts at 2.5 and moves −0.20 / −0.15 / 0 / +0.15 by answer, floored at 1.3.
- *Again* on a review card is a lapse: ease drops, the card goes to relearning for 10 minutes and comes back at half its old interval.
- Intervals over 4 days get ±5% fuzz so batches don't clump on one day.
- Daily caps on new cards and reviews, reset at local midnight.

The interval printed on each answer button is produced by running the real scheduler in dry mode, so what the button says is exactly what happens.

## Data

Cards sync to a Supabase project and follow the signed-in user across devices. Study settings (new cards/day, review direction, etc.) stay local to each device. Export a JSON backup from the **Desk** tab any time. The TSV export imports into Anki as three fields: Word / Meaning / IPA.

## Sign-in & sync setup (one-time, in Supabase)

1. **Run the table script** in the SQL Editor — builds the `cards` table and the row-level security policy so each person only ever sees their own cards.
2. **Set Site URL + Redirect URLs.** In the Supabase dashboard: *Authentication → URL Configuration* → set **Site URL** to your live Pages URL (e.g. `https://vocario.pages.dev`), and add that same URL under **Redirect URLs**. Without this, the magic-link email won't send people back to the app.
3. The publishable key is already in `index.html` — it's meant to be public. The actual protection is the row-level security policy from step 1, not the key being secret.

Cards created before sync was added, if any, are uploaded automatically the first time you sign in on that device.
