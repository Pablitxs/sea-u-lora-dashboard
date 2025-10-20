# SEA U LO-RA Dashboard (Electron)

Desktop Electron app for the SEA U LO-RA Dashboard. It wraps a static HTML UI, integrates a LILYGO T‑Beam (MASTER) via serial port for real‑time SOS parsing, and renders an offline map from a packaged PMTiles archive.

## Features
- Electron desktop app (Windows) loading the HTML dashboard (no external web server)
- Offline map via PMTiles (packaged `zambales.pmtiles` with a custom `pmtiles://` protocol)
- Serial integration with the MASTER device (LILYGO T‑Beam SX1262 915MHz)
  - Auto parsing of SOS messages (single-line and multi-line block formats)
  - Persisted SOS history; dashboard aggregation per device; map deep‑linking to coordinates
- Local settings (dark mode, language) and login credentials stored in localStorage
- Windows installer/portable builds via electron-builder

## Quick start
Prerequisites:
- Node.js LTS (18+; 20 recommended)
- Windows (build scripts here target Win x64)

Install and run:
```powershell
# from this folder
npm ci   # or: npm install
npm start
```
This launches Electron and opens `login.html`.

Default login:
- Username: `admin`
- Password: `admin123`

Change credentials any time via: `settings.html` → “Change Login Credentials”.

## Build and packaging
App icons (run once or when logo changes):
```powershell
npm run build:icons
```
Portable executable (dist/…Portable.exe):
```powershell
npm run build:win
```
NSIS installer (dist/…Setup.exe):
```powershell
npm run build:installer
```
Other:
```powershell
npm run build        # cross‑platform defaults from electron-builder
npm run build:dir    # unpacked directory build (dist/win-unpacked)
```
Notes:
- Code signing is disabled in scripts for convenience (`ELECTRON_BUILDER_DISABLE_CODESIGNING=true`).
- Outputs are written to `dist/`.

## Project layout (top level)
```
.
├─ electron/                 # Electron main & preload
│  ├─ main.js
│  └─ preload.js
├─ assets/                   # Leaflet, CSS, etc.
├─ Pmtiles_viewer-main/      # PMTiles loader for the Map page
├─ fontawesome-free-*/       # Icons
├─ *.html                    # login, dashboard, map, settings, sos pages
├─ zambales.pmtiles          # Offline map archive (packaged)
├─ package.json              # scripts, deps, electron-builder config
└─ package-lock.json
```

## How it works
- Entry point: `electron/main.js` loads `login.html` and exposes safe IPC via `electron/preload.js`.
- Serial: Uses `serialport` to read lines from the MASTER device. SOS events are parsed and broadcast to renderers; history is persisted.
- Offline map: `map.html` loads `pmtiles://zambales.pmtiles` through a custom Electron protocol for Range requests; Leaflet renders either rasterized tiles or vector (depending on archive).
- UI pages:
  - `SEA U LO-RA DASHBOARD.html`: overview, devices/fishermen lists, live SOS panel
  - `sos-log.html` / `sos-detail.html`: per‑device aggregation and detail timeline
  - `map.html`: offline map with info cards (SOS and Master)
  - `settings.html`: dark mode, language, serial COM selection, credentials

## Data & persistence
Electron user-data directory (Windows example):
- Typically: `%APPDATA%/SEA U LO-RA Dashboard/`

Files stored there by the app:
- `serial-config.json` — latest COM port and baud
- `master-gps.json` — last known MASTER GPS snapshot
- `sos-log.json` — persisted SOS history (up to 1000 entries)

UI preferences stored in the renderer (localStorage):
- Theme (`ui.theme`), language (`ui.lang`)
- Dashboard lists and counters
- Login state and credentials (`auth.*`, `auth.cred.*`)

## Using the MASTER (serial) features
1) Open `settings.html` inside the app.
2) Click “Refresh” under MASTER Device Connection to list COM ports.
3) Select your MASTER’s COM port and set baud (default 115200).
4) Click “Connect” and optionally “Save” to persist the selection.
5) Incoming SOS lines are parsed and shown in Dashboard/SOS Log/Map.

If the map does not show tiles:
- Ensure `zambales.pmtiles` is present in the app root.
- For packaged builds, it is included via electron-builder `files` and accessed with `pmtiles://`.

## Scripts (from package.json)
- `start` — launch Electron
- `build:icons` — generate icons from `NEW LOGO.jpeg`
- `build:win` — Windows portable
- `build:installer` — Windows NSIS installer
- `build`, `build:dir`, `pack:win` — additional build/pack options

## Troubleshooting
- No COM ports listed: install the correct USB‑serial driver for your board (e.g., CP210x/CH34x) and reconnect.
- “Map is unavailable”: verify `zambales.pmtiles` exists; restart the app after adding/replacing the file.
- Coordinates missing/invalid: the map will not navigate for SOS entries without valid latitude/longitude.

## License
MIT (see `package.json`).

## Acknowledgements
- Electron, electron‑builder, electron‑packager
- serialport
- Leaflet
- PMTiles / Protomaps
- Font Awesome
