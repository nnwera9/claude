# The Newsstand

A single-file, zero-build news reader that pulls live RSS from seven newsrooms —
Financial Times, Wall Street Journal, New York Times, Bloomberg, The Economist,
The New Yorker, and Reuters — into one editorial front page.

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
  a second relay is launched after ~2.4s and the first valid response wins. A
  slow or blocked relay can no longer stall a feed — this is what removes the long
  FT/WSJ delays.
- **Direct-link preference.** When the same story arrives from a publisher feed
  (clean URL) and Google News (redirect), the clean publisher URL is kept.
- **Health-scored relays with memory.** Relays are ranked by a Laplace-smoothed
  success rate and the last working relay per feed host is remembered across visits.
- **Instant paint from cache.** The last session's articles render immediately,
  then refresh underneath. Sources stream in progressively as they resolve.

Clicking any headline opens the article **at the source in a new tab** — there is
no interstitial preview.

## Features

- Dark "evening" / light "morning" editions
- Cross-outlet story clustering ("Tracking")
- Source and topic filters, search, newest/oldest/by-source sort, comfortable/compact density
- Bookmarks (export to Markdown), read tracking, hide-read
- Live market tape (Yahoo Finance) and an embedded live-TV player (official network YouTube streams)
- Full keyboard navigation (press `?` for the shortcut list)

## Notes

If *every* source fails to load, it's almost always a privacy extension/ad-blocker
blocking the CORS relays, or a shared VPN exit exhausting their rate limits.
The in-app diagnostics (⋯ menu → "Show relay diagnostics") show per-relay health.
