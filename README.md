# The Newsstand

A single-file, zero-build news reader that pulls live RSS from nine newsrooms —
Financial Times, Wall Street Journal, New York Times, Bloomberg, Reuters,
BBC News, AP News, The Economist, and The New Yorker — into one editorial
front page.

Open `index.html` in any modern browser. No server, no install. Reading history
and bookmarks stay in `localStorage` on your device.

## How the feed engine works

The page runs entirely in the browser, so it can't fetch publisher RSS directly
(those servers don't send CORS headers, and premium publishers actively block
datacenter/proxy IPs). The engine is built around that reality:

- **Parallel publisher + aggregator fetch.** Each source pulls its own RSS feeds
  *and* a Google News query at the same time, then merges and de-dupes. Publishers
  that block proxies (FT, WSJ, Bloomberg) always have a working path through Google
  News, and coverage is much wider than a single feed.
- **Hedged relay racing.** Instead of waiting out a long timeout on one CORS relay,
  a second relay is launched after ~2.2s and the first valid response wins. A
  slow or blocked relay can no longer stall a feed.
- **VPN / rate-limit awareness.** A relay answering HTTP 429 (per-IP rate limit —
  typical on shared VPN exit IPs) is benched for 90 seconds so the race stops
  feeding it. Feed fetches are staggered and capped at 6 concurrent.
- **Direct-link preference.** Google News redirect tokens are base64-decoded
  client-side where possible so clicks land directly on the publisher; when the
  same story arrives from a publisher feed and an aggregator, the clean publisher
  URL is kept.
- **Health-scored relays with memory.** Nine relays (two feed-to-JSON converters
  plus seven generic CORS proxies) are ranked by a Laplace-smoothed success rate;
  the last working relay per feed host is remembered across visits.
- **Instant paint from cache.** The last session's articles render immediately,
  then refresh underneath. Sources stream in progressively as they resolve.

Clicking any headline opens the article **at the source in a new tab** — there is
no interstitial preview.

## Features

- Article images parsed from `media:content` / `media:thumbnail` / enclosures / inline HTML
- Settings panel: enable/disable sources, image toggle, auto-refresh cadence (3/5/10/15 min)
- Dark "evening" / light "morning" editions
- Cross-outlet story clustering ("Tracking") and "new since last visit" markers
- Source and topic filters, search, newest/oldest/by-source sort, comfortable/compact density
- Bookmarks (export to Markdown), read tracking, hide-read, share (Web Share API)
- Live market tape (Yahoo Finance) and an embedded live-TV player (official network YouTube streams)
- Full keyboard navigation (press `?` for the shortcut list), back-to-top

## Notes

If *every* source fails to load, it's almost always a privacy extension/ad-blocker
blocking the CORS relays, or a shared VPN exit exhausting their rate limits — the
reader detects repeated 429s and says so. The in-app diagnostics (⋯ menu → "Show
relay diagnostics") show per-relay health, active cooldowns, and last errors.
