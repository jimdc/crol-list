# CROL-List — Plan: mockups → real features

CROL-List is **static, client-side, live-data** (see README → Architecture). Most of it is real. A
few pieces are honest mockups for the demo. This plan is what each needs to become real, and the
order to do it.

## The one architectural fork

The site is 100% static (no backend, no keys) — that's why it runs on GitHub Pages. Two features
need a **thin backend**; everything else can stay client-side:

- a tiny **scheduled worker** (launchd/cron or cron-triggered serverless fn) → real **alert delivery**
- a tiny **API-key proxy** (serverless fn) → real **NL model**

## Mockup → real

| # | Mock today | To make it real | Backend? | Effort |
|---|---|---|---|---|
| 1 | **Alerts don't send** — preview only; "Save" = localStorage | Scheduled job re-runs the saved query, diffs vs last run, sends new items via **Apprise/Listmonk** (email) or an SMS gateway; persist subscribers | **Yes** | L |
| 2 | **NL→boolean** is an on-device regex parser | Swap to a **Claude Haiku** call (server-side, key-held) that decomposes NL → SODA filters; apply *all* constraints | **Yes** | M |
| 3 | **Land "near 79 Rivington"** uses a hardcoded alias | **Geocode** the address (GeoSearch — wired) → BBL/block + neighbourhood → query **ZAP-BBL (`2iga-a6mk`)** by block; pin the real address; honest "nothing on this lot, nearest in <neighbourhood>" fallback | No | M |
| 4 | **Land map pin** = approximate geocode of the project *name* | ZAP-BBL → real BBLs → **MapPLUTO polygons** (DCP ArcGIS FeatureServer) → draw the actual rezoned lots | No | M |
| 5 | **Money shows awarded $ only** (City Record, $5B outlier cap) | Join **Checkbook NYC** by PIN → dollars **actually paid** + vendor track record + M/WBE | No (CORS) | M |
| 6 | **Dateline "Vol. CLIII · No. 120"** was hardcoded | ✅ now computed (Vol = year−1873 in Roman; No. = weekdays elapsed) + a footer note that we span every edition | No | done |
| 7 | **City seal** is a stylized SVG placeholder | Drop in the real NYC arms SVG | No | S |
| 8 | **"Find in City Record" links are generic search** (rezone/alert items) | Resolve the *specific* notice: query `dg92-zbpx` by ULURP number / PIN → deep-link to `RequestDetail/{request_id}` (procurement already does this; extend to land + alerts) | No | M |

## Robustness: cache a snapshot as a fallback (recommended for demo day)

Live-API is best for *freshness*, but the demo depends on NYC Open Data being up + fast at runtime.
Add a small **cached snapshot** in `data/` as a **fallback + fast first paint**:

- Commit `data/recent_open_rfps.json`, `data/recent_rezonings.json`, `data/recent_awards.json`
  (a few dozen rows each, refreshed by a one-line script before the demo).
- On load, render from the snapshot **instantly**, then replace with the live fetch when it returns;
  if the live call fails or times out, **keep the snapshot** and show a small "showing cached <date>" tag.
- Keep live as the default for everything else (freshness). This is the *only* place a cache earns its keep.

## Already real (keep)

Live City Record / payroll / ZAP queries · PIN chaining · person→payroll join (~98.6% match) ·
CSV export · `.ics` calendar · `mailto`/`tel` ✍ Respond · the title crosswalk data.

## Recommended order

1. **#3 + #4 (Land, no backend)** — biggest visual payoff, removes the two most obvious fakes.
2. **Cache fallback** — cheap insurance for demo day.
3. **#5 Checkbook** — strongest "follow the money" upgrade, client-side.
4. **#1 alert delivery** — the headline "it actually emails you"; scope deliberately (email-only, daily cron, simple subscriber store).
5. **#2 real NL**, then **#7** polish.

## More of the journal to make usable (new lenses)

CROL-List lenses 3 of the City Record's sections today. The same `dg92-zbpx` data has more, worth surfacing (live section counts):

| Section | ~Rows | Lensed? | Feature idea |
|---|---|---|---|
| Changes in Personnel | 955K | ✅ People | — |
| Procurement | 104K | ✅ Money | — |
| Contract Award Hearings | 10.5K | — | "Speak before it's signed" — the public-comment window on a contract + hearing date (.ics) |
| Public Hearings & Meetings | 8.9K | ◑ Land (CPC only) | **Meetings-near-you calendar** beyond rezonings — Community Board, Comptroller, Housing Authority, BERS — filter by borough/agency, add to calendar |
| Special Materials | 8.1K | — | HPD + agency special notices |
| Agency Rules | 3.0K | — | **"What rules are changing"** — plain-English digest of proposed/adopted city rules, by agency, with the comment deadline (high civic-engagement value) |
| Public Comment on Contract Awards | 783 | — | folds into Contract Award Hearings above |
| **Property Disposition** | 243 | — | **The original hook** — city property sold / auctioned / disposed. "What's being sold near you" + the demolition cross-ref (DOB). Tiny dataset, vivid |
| Court Notices | 155 | — | legal-notices feed |

**Highest value-per-effort:** **Property Disposition** (small, vivid, the founding idea) and **Agency Rules**
(real civic utility — tell people which rules are changing and when the comment window closes).
