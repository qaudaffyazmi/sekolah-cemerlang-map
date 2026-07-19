# Peta Sekolah Kecemerlangan Malaysia (SBP & MRSM)

Interactive map of Malaysian excellent schools (Sekolah Berasrama Penuh & MRSM)
with SPM GPS performance, distance-from-home, fuel cost estimates, and EV
chargers along the route.

**Live site:** https://YOUR-DOMAIN-HERE.com
**Stack:** Single static HTML page + Leaflet + OpenStreetMap. No build step, no
backend, no database. Hosted on Cloudflare Pages (free, unlimited bandwidth).

---

## Repository structure

```
├── index.html         The entire app (HTML + CSS + JS in one file)
├── schools-data.js    School data — THIS is the file you edit for updates
├── schools.json       Same data in pure JSON (reference / tooling; not loaded by the site)
└── README.md          This file
```

`index.html` loads `schools-data.js` via a `<script>` tag, so both files must
stay in the repo root together.

---

## How to update the data

1. Open `schools-data.js` in the GitHub web editor (press `.` on the repo page,
   or click the file → pencil icon).
2. Edit the values (see field reference below).
3. Commit to `main`. Cloudflare Pages redeploys automatically — changes are
   live worldwide in ~60 seconds.

No tools to install, no build to run. If you make a syntax mistake (missing
comma, unclosed quote) the map will load empty — open the browser console
(F12) to see the error, or revert the commit.

### Annual SPM results update

For each school, add the new year inside `gps` and update `purata`:

```js
"gps": { "2023": 1.25, "2024": 1.05, "2025": 1.69, "2026": 1.31 },
"purata": 1.303,
```

Year labels, sparklines, and the compare table adapt automatically — no code
changes needed.

---

## Field reference (one object per school)

| Field | Type | Example | Notes |
|---|---|---|---|
| `id` | string | `"tkc"` | Unique, lowercase, no spaces. Never reuse. |
| `name` | string | `"Kolej Tunku Kurshiah"` | Full name, no acronym. |
| `acronym` | string | `"TKC"` | Shown in lists/popups. If the school has none, repeat the name. |
| `pin` | string | `"MRSM SAS"` | Short label inside the map pin. Keep under ~12 chars. |
| `type` | string | `"SBP"` or `"MRSM"` | Controls pin colour and filters. Exact spelling. |
| `program` | string | `"SMS"` | SMS, SBPI, IGCSE, Premier, U. Albab, etc. |
| `jantina` | string | `"P"`, `"L"`, `"LP"` | P = perempuan, L = lelaki, LP = both. |
| `hot` | number/string/null | `1` | HOT group; `"MRSM"` for MRSM rows in the source sheet. |
| `kemasukan` | string | `"1,4"` | Intake forms. |
| `phone` | string | `"BP Weekend"` | Free text. Use `"—"` if unknown. |
| `ads` | boolean | `true` | Apple Distinguished School. |
| `founded` | number/null | `1947` | |
| `negeri` | string | `"Negeri Sembilan"` | Full state name (drives the filter dropdown). |
| `calon2021` | number/null | `137` | SPM candidates 2021. |
| `purata` | number | `1.294` | Average GPS. **Lower = better.** Drives default sort. |
| `lat`, `lng` | number | `2.729322` | Decimal degrees. Right-click the campus in Google Maps → first menu item copies these. |
| `gps` | object | `{"2023":1.25,...}` | Year → GPS. Omit years with no data (do NOT write `"NA"`). |

### Adding a new school

Copy an existing object, change every field, give it a fresh `id`. Verify the
coordinate lands on the actual campus, not the town centre — this is the most
common data error.

---

## External services used (all free)

| Service | Used for | Key needed? | Limits / notes |
|---|---|---|---|
| OpenStreetMap tiles | Base map | No | Fine for launch. If traffic gets heavy, swap the tile URL to Carto or MapTiler (one line in `index.html`). |
| Nominatim | "Cari alamat" home search | No | Policy: light use only. Swap to another geocoder if abused. |
| OSRM demo server | Driving routes | No | Demo server, no SLA. **Before heavy traffic:** switch to OpenRouteService (free key, 2,000 routes/day) or self-host OSRM. The call is isolated in `selectSchool()` / `getRoute()`. |
| OpenChargeMap | EV chargers | Recommended | Register free at openchargemap.org/site/develop/api and paste into `OCM_API_KEY` at the top of the script in `index.html`. |

**Note on keys:** anything placed in these files is publicly visible (static
site). That's acceptable for the free public-data keys above. Never put a
billing-enabled key (e.g. Google Maps) here — if that's ever needed, proxy it
through a Cloudflare Pages Function with the key in an environment variable.

---

## Deployment

Hosted on **Cloudflare Pages**, connected to this repo.

- Build command: *(empty)* · Output directory: `/`
- Every push to `main` auto-deploys.
- Custom domain is configured under the Pages project → **Custom domains**.
- Analytics: Cloudflare → Analytics & Logs → Web Analytics.

To roll back a bad deploy: Pages project → **Deployments** → pick a previous
deployment → **Rollback**.

---

## Data caveats (keep these honest)

- Source data was extracted from screenshots of SBP/MRSM lists (Jul 2026).
  **Verify against official KPM / MARA sources** before relying on it.
- Coordinates were manually collected; spot-check pins against real campuses.
- Fuel prices: BUDI95 (subsidised) is stable, but the market price changes
  **every Wednesday** under APM — the site lets users edit it, and the default
  in `index.html` should be refreshed occasionally.
- EV charger data comes from OpenChargeMap (community-maintained); Malaysian
  coverage is good but not complete.
- List cards show **straight-line** distance ("≈ km lurus"); only the route
  card and compare table show true driving distance.
