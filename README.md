# CROL-List

**NYC's official record (The City Record), made browsable.** The classifieds of city government.

[The City Record](https://a856-cityrecord.nyc.gov/) is the official daily journal of the City of
New York — by law (City Charter §1066) every agency must publish its contracts, personnel changes,
hearings, and rezonings there. It's the city's daily newspaper of government, and almost nobody can
read it: a dense stream of disconnected legal notices. **CROL-List** re-links those notices into
something a person can actually follow.

**Live (no login):** https://jimdc.github.io/crol-list/

## What it does — seven lenses

**Three deep lenses** re-stitch a single thread and decode it:

- **💵 Money** — follow a contract from **RFP → Intent to Award → Award ($)**, stitched by PIN, with how-to-respond (deadline, contact, PASSPort), filters, and CSV export. Plus a plain-English ("ask in English") search.
- **👤 People** — decode any city job: official civil-service title, **competitive (exam) vs non-competitive**, salary band, and a career ladder. Or look up a person → appointment history + payroll.
- **🏗 Land** — rezonings in plain English, cross-referenced to **ZAP** (applicant, what's being built, affordable housing, status) and drawn as the real rezoned tax-lot polygons on a map.

**Three feed lenses** sweep the rest of the daily record — filter by agency or keyword, then add the date to your calendar (`.ics`):

- **🏛 Property** — city property being **sold, auctioned, or disposed** (NYPD auctions, HPD sales, DEP land): what's on the block, the sale date, and where. Each address gets a one-tap **"Still standing?"** check against DOB demolition filings.
- **📋 Rules** — **rules that are changing**: proposed & adopted agency regulations, by agency, with the public-comment **hearing date**.
- **🗓 Meetings** — **public meetings**: Community Boards, City Council, Landmarks, Board of Standards & Appeals, and more.

**And one to keep it coming:**

- **🔔 Alerts** — subscribe to a slice (e.g. "rezonings near 79 Rivington," "awards over $1M") and preview the email/SMS digest, with one-tap **✍ Respond** / **✉** / **☎** built from the notice's own data.

## Architecture — client-side, live data, no backend

CROL-List is **one self-contained `index.html`**: inline CSS + vanilla JS, **no build step, no server,
no API keys**. It opens with a double-click and is hosted on GitHub Pages as static files.

**Every query is a live API call from the browser at runtime.** There is no cached bulk data, no
scheduled download, and no copy of the datasets in this repo — so the data is always as fresh as the
City publishes it (the City Record updates each business day). The browser calls, directly:

```
 ┌────────────── index.html (static, on GitHub Pages) ──────────────┐
 │  vanilla JS  ──fetch()──►  NYC Open Data / Socrata SODA API       │
 │                            • City Record   dg92-zbpx (daily)      │
 │                            • Citywide Payroll  k397-673e          │
 │                            • Civil Service List  vx8i-nprf        │
 │                            • ZAP projects  hgx4-8ukb              │
 │                            • DOB filings  w9ak-ipjd / ic3t-wcy2   │
 │              ──fetch()──►  Planning Labs GeoSearch (geocoding)    │
 │              ──query───►  MapPLUTO (ArcGIS) tax-lot polygons      │
 │              ──tiles───►  Leaflet + CARTO basemap                 │
 └──────────────────────────────────────────────────────────────────┘
```

This works because those APIs are **CORS-open and need no key**. The trade-off: the demo depends on
those services being up at runtime (they are), and a few endpoints are rate-limited rather than keyed.

The **only** data committed to the repo is two small *precomputed snapshots*, used as seed/reference
(the People tab still computes its live results on the fly):

- `data/title_crosswalk.json` (~250 roles) and `data/people_examples.json` (~16 seed roles), with method notes.

## Data sources

| Source | ID / endpoint | Used by |
|---|---|---|
| City Record Online | `dg92-zbpx` (Socrata) | Money · People · Property · Rules · Meetings · Alerts |
| Citywide Payroll | `k397-673e` | People |
| Civil Service List | `vx8i-nprf` | People (exam status) |
| ZAP Projects | `hgx4-8ukb` | Land, Alerts |
| Planning Labs GeoSearch | `geosearch.planninglabs.nyc` | Land, Property (geocoding) |
| MapPLUTO (ArcGIS) | `services5.arcgis.com/…/MAPPLUTO` | Land (tax-lot polygons) |
| DOB job filings | `w9ak-ipjd`, `ic3t-wcy2` | Property ("Still standing?") |
| Checkbook NYC | `checkbooknyc.com/api` | *(roadmap — actual $ paid)* |

## Real vs. mock

Most of the app is **real** (live queries, working CSV/.ics/mailto/tel). A few pieces are **honest
mockups** for the demo — the alert *delivery*, the natural-language *model*, and the address→project
*proximity*. What each needs to become real is written up in **[`PLAN.md`](PLAN.md)**.

## Run it

```
open index.html      # double-click — no server, no build, no keys
```
