# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **Daily Car Balance Tracker** for Mongla Port Authority's Traffic Department — a zero-dependency, offline-first PWA that tracks daily vehicle movements across 8 port locations. There is **no build system, no package manager, no test framework**. The app is plain HTML/CSS/vanilla JS that runs directly from `file://` or from Cloudflare Pages.

## Commands

```bash
# "Run" the app — just open it
xdg-open index.html        # or open in any browser

# Syntax-check before committing (there are no tests; this is the safety net)
node --check src/app.js
node --check worker/worker.js
node --check service-worker.js
```

There is nothing to install or compile. Edits to `src/app.js` / `index.html` / `src/styles.css` take effect on a browser reload.

## Layout

```
index.html  manifest.json  service-worker.js   # must stay at repo root
src/      → app.js, styles.css                 # everything index.html loads
assets/   → car.png
worker/   → worker.js                          # separate Cloudflare deploy
docs/     → SETUP-GITHUB-SYNC.md
```

If you move an asset, update **both** its `index.html` reference and the `STATIC_ASSETS` list in `service-worker.js` (paths must match), then bump `CACHE_NAME`. `service-worker.js` and `index.html` cannot move from root (SW scope / entry point).

## Critical conventions (these cause real bugs if missed)

- **Bump the service worker cache on every code change.** `service-worker.js` is cache-first for `src/app.js`, `index.html`, `src/styles.css`. If you change any of them without incrementing `CACHE_NAME` (`car-balance-vN`), installed PWAs keep serving stale code. Always bump it and tell the user to hard-reload (Ctrl+Shift+R).
- **Date storage vs display are separate.** Dates are stored and used as lookup/sort keys in `YYYY-MM-DD` form (produced via `toLocaleDateString("en-CA", …)`). **Never** change those to another format — DB keys, `sett.hols`, `sett.transfers`, and `isRed()` all depend on it. For anything the user *sees*, format with `fmtDMY()` → `dd-mm-yyyy`. Manual date entry fields use `dd/mm/yyyy` text inputs backed by helpers `dmyToISO` / `isoToDMY` / `dmyMask` / `dmyPick` (native `<input type=date>` can't be forced to a locale, hence the text-input approach).
- **Treat all stored data as untrusted.** Cloud data can arrive from other devices, so run user-supplied strings through `esc()` before inserting into HTML. Load paths also self-heal known corruption (duplicate dates, numeric-keyed objects coerced back to arrays).
- **`saveToFirebase` / `loadFromFirebase` are misnomers.** Firebase was removed; these now talk to the Cloudflare Worker. Don't reintroduce Firebase.
- Commits go **directly to `master`** (no PR flow in this repo's history).

## Data & sync architecture

Three moving parts across **two GitHub repos** plus a Worker:

- **`carview` (this repo)** → app code → deployed to Cloudflare Pages (`carview.pages.dev`).
- **`carview-data` (private repo)** → holds a single `data.json` → the actual user data. **Every save is one commit**, which is what powers the in-app Version History / restore.
- **`worker/worker.js`** → a Cloudflare Worker (`carview-proxy.*.workers.dev`) deployed **separately** (it is not loaded by the app). It proxies reads/writes to `carview-data` so the GitHub token never reaches the browser. Endpoints: `GET /` (current data), `GET /?history=1`, `GET /?at=<sha>`, `PUT /` (commit). Worker secrets: `GITHUB_TOKEN`, `WRITE_PASSWORD`. See `docs/SETUP-GITHUB-SYNC.md`.

Client side:
- `GITHUB_CONFIG.workerUrl` in `src/app.js` points at the Worker; the same URL must be in the `connect-src` of the CSP `<meta>` in `index.html`.
- All persisted state is one object: `{ db: DB, sett, users, loggedIn, adminHash }`. Local copy lives in `localStorage` under key `carbal_v7`; cloud copy is `data.json`.
- **Overwrite protection:** the client tracks `cloudBaseSha` and the Worker returns `409` if the file changed since load, so concurrent edits don't clobber each other.
- Save feedback distinguishes "✓ Saved to cloud!" from "⚠ Saved to device only!" — the latter means the Worker write failed (e.g. expired token) and only `localStorage` has the data.

## In-memory data model (`src/app.js`)

- `DB[monthKey]` where `monthKey` is `"YYYY-MM"`; value is an array of day rows:
  `{ date:"YYYY-MM-DD", del:[8], imp:[8], bal:[8], ob:[8]|null, al, av, rn }`
  The `8` matches `LOCS` (8 port locations); `del`=deliveries, `imp`=receipts/imports, `bal`=balances, `ob`=stored opening balance, `av`=auction value, `rn`=rotation no.
- `sett` = `{ fri, sat, sun, hols:[], excs:[], transfers:{}, tz, … }`.
  - `sett.transfers["YYYY-MM-DD"] = [{from, to, qty, note}]` — inter-location car moves; balances are recomputed by `applyTransfersToRow()` and `recalcTransferCascade()`, which **cascade opening balances across subsequent months**.
  - `isRed(date)` decides "red"/off days with strict precedence: **`excs` (force working day, always wins) → weekly toggle (`fri`/`sat`/`sun`) → `hols`**.
  - `BD_HOLIDAYS` (per-year bundled holidays) + `BD_FIXED` (auto-generated fixed-date national holidays) feed the Bangladesh holiday loader (`loadBDHolidays`).
- `users` = `{ username: passHash }` (max 3 + admin); `ADMIN_USER`/`ADMIN_HASH`; auth uses SHA-256 via Web Crypto. `writeAuth` (in `localStorage`) is the key the Worker checks for PUTs.

## UI / rendering flow

- Single page with tabs switched by `showPage(p, el)` → `daily | chart | report | settings | transfer`; each tab lazily calls its render function (`renderTable`, `renderCharts`, `renderReport`, `renderSettings`, `renderTransferPage`).
- Charts use **Chart.js** (CDN, SRI-pinned); always create them through `safeChart()` and tear down with `killCharts()` to avoid leaks. Excel export uses **SheetJS / XLSX** (CDN). The app degrades gracefully if CDNs are unreachable.
- `setDirty(true)` after any data mutation triggers auto-save; `requireLogin(cb)` gates edits; `showError` / `showSuccess` / overlays handle user feedback.
- Keyboard shortcuts are defined in `SHORTCUTS` (Ctrl+S save, Ctrl+Z undo, 1–4 tabs, etc.).

## Language for user-facing text

The user prefers **Banglish (Bengali in Latin script), never Bangla script** — applies to UI strings, alerts, and replies.
