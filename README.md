# CivicPulse

**Better Cities Begin With Us.**

CivicPulse is a civic-tech platform that connects citizens with government departments to report, track and genuinely resolve civic problems in Bengaluru. Its core rule: **a complaint is closed only when the citizen who reported it confirms it is fixed.** A department can never close a complaint by marking it resolved.

![CivicPulse home page](docs/screenshots/home.png)

## Citizen-verified resolution

1. A citizen reports a problem with photos, a map pin, a category and who is affected.
2. The department reviews it and assigns a maintenance team.
3. The team works on it; progress is public.
4. The department uploads after-repair photos and a description.
5. The status becomes **Awaiting Citizen Verification** and the reporter is notified.
6. The reporter chooses **Yes, fixed** (closed, confirmation date recorded) or **No, not fixed** (reopened with a reason and fresh photos; earlier repair records are kept).

If the reporter doesn't respond before the administrator's deadline, the complaint is closed as *not verified by citizen* and never counted as citizen-confirmed. Anonymous reporters keep the power to verify through a one-way key derived from their account.

| Citizen verification | Department dashboard |
|---|---|
| ![Verification](docs/screenshots/verify.png) | ![Department dashboard](docs/screenshots/gov.png) |

## Features

- **Three roles:** Citizen, Government Official (per department), Administrator
- **Reporting:** photos and video, map pinning, English and Kannada, voice input, anonymous mode, unique complaint IDs
- **Civic map:** status-coloured markers, filters, heatmap and a labelled predicted-risk layer
- **Government portal:** priority queue, team assignment, repair evidence, dispute replies, analytics
- **Accountability:** automatic escalation of overdue complaints, reopen tracking, full audit timeline, community verification by nearby residents
- **AI:** photo and text classification and before/after repair comparison (Claude); suggestions only, a person decides
- **Civic intelligence:** duplicate detection, priority scoring, Civic Health Scores with published formulas, recurring-problem predictions, repeat-location detection, unheard-neighbourhood detection
- **Civic SOS:** one-tap hazard reports (exposed wires, fallen poles, open manholes, fallen trees, flooding)
- **Accessibility navigator:** resident-verified ramps, crossings and obstacles with a route check
- **Community:** volunteer drives with certificates, Civic Challenge leaderboard, NGO resource exchange
- **Public money transparency:** projects shown only with a linked public source; no invented figures

## Technology

| Layer | What is used |
|---|---|
| Frontend | HTML5, CSS3, JavaScript, Leaflet 1.9.4, Chart.js 4 |
| Algorithms | C (`c/civic_engine.c`): severity, priority, location duplicate search, image-hash distance. Compiled to WebAssembly for the browser and to a shared library for Python (ctypes). A JavaScript port is the fallback. |
| Data and auth | Claude artifact runtime (shared database, role-based write rules, claude.ai sign-in), or browser localStorage in demo mode |
| AI | Anthropic Claude via the artifact runtime's `sample` capability |

## Repository layout

```
index.html            Built single-page app (open or host this)
src/                  App source: core.js, local.js (demo mode), analysis.js, ui.js, views*.js, app.css, localities.json
c/                    C engine source and compiled civic_engine.wasm
build.py              Bundles src/, the WASM engine and Leaflet CSS into index.html
scripts/              gen_map.mjs (neighbourhood boundaries), seed.mjs (labelled sample data)
data/seed.json        Sample data (every record is marked as SAMPLE)
tests/                C engine unit tests (ctypes) and end-to-end browser tests (Playwright)
docs/                 Screenshots and the project presentation
```

## Build and run

Requirements: Node.js 18+, Python 3.10+, clang (for WebAssembly) and gcc.

```bash
npm install                      # leaflet, chart.js, d3-delaunay
python3 build.py                 # writes index.html
```

Rebuild the C engine:

```bash
# WebAssembly, used by the web app
clang --target=wasm32 -O2 -nostdlib -Wl,--no-entry -Wl,--export-dynamic -o c/civic_engine.wasm c/civic_engine.c
# Native shared library, used by Python via ctypes
gcc -O2 -shared -fPIC -o c/libcivic_engine.so c/civic_engine.c
```

Regenerate the neighbourhood map or the sample data:

```bash
node scripts/gen_map.mjs
node scripts/seed.mjs
```

## Two ways to run it

- **GitHub Pages (demo mode):** `index.html` runs on any static host. Outside Claude it switches to demo mode: data is saved only in the visitor's browser (localStorage), preloaded with the labelled sample data, with real OpenStreetMap street maps and device GPS. Every role can be tried with "Viewing as". AI suggestions are not available in this mode.
- **Claude artifact (shared mode):** published as a Claude artifact, the same file uses a shared realtime database, claude.ai sign-in and Claude AI.

To publish on GitHub Pages: repository **Settings → Pages → Build and deployment → Deploy from a branch → `main` / `(root)` → Save**. The site appears at `https://<username>.github.io/civicplus/`.

## Tests

```bash
gcc -O2 -shared -fPIC -o c/libcivic_engine.so c/civic_engine.c
python3 tests/test_engine.py     # C engine unit tests

pip install playwright && playwright install chromium
python3 tests/run.py             # end-to-end: report → review → assign → repair → dispute → reopen → repair → confirm
python3 tests/shots.py           # regenerates docs/screenshots
```

The browser tests run the app against an in-memory mock of the database (`tests/mock.js`).

## Limitations

- Not connected to any official government system; department names are working names.
- Neighbourhood boundaries are approximate. Inside the Claude artifact host, street tiles and device GPS are blocked, so the map is schematic and locations are pinned by hand; on GitHub Pages both work.
- In demo mode data stays in one browser and is not shared between visitors.
- Weather is not fetched automatically; administrators enter official rain warnings.
- Notifications are in-app only (no email or SMS).
- Department-level permissions for officials are enforced by the app, not the database.

## Roadmap

- FastAPI + PostgreSQL backend with the C modules loaded through ctypes
- Deployment on Vercel (frontend) and Render (API)
- SMS and WhatsApp notifications, integration with municipal complaint systems
- A pilot in one Bengaluru ward to measure resolution time and repeat complaints

## Emergencies

CivicPulse does not replace emergency services. Call **112** in an emergency, **1912** for BESCOM electrical hazards and **1533** for the BBMP helpline.
