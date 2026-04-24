# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development

This is a zero-dependency, no-build-step static PWA. Serve locally with:

```bash
python3 -m http.server 8080
```

Must be served over HTTP (not opened as a file) for the service worker to register. No npm, no bundler, no tests, no linter.

## Architecture

The entire application lives in a single `index.html` (≈2,800 lines) alongside `sw.js` (service worker) and `manifest.json`. There is no build process, no `src/` directory, and no dependencies — external resources are CDN-only.

### Key files

- `index.html` — All HTML, CSS, and JavaScript in one file
- `sw.js` — Service worker: caches app shell and API responses for offline use
- `manifest.json` — PWA manifest (icons, display mode, theme color)

### JavaScript structure (inside `index.html`)

The JS is split into two IIFE blocks:

**Chunk 1 (lines ~1576–2038): API + state + search**
- `geocode(query)` — calls Open-Meteo geocoding API
- `fetchWeather(lat, lon)` — calls Open-Meteo forecast API
- `loadCity(place)` — orchestrates fetch → render cycle
- `renderHourly()` / `renderDaily()` — build forecast HTML strings
- `boot()` — restores last city from localStorage or defaults to Brisbane
- State is held in module-level variables: `lastData`, `lastPlace`, `searchAbort`, `weatherAbort`
- Unit toggle (°C/°F) re-renders from `lastData` without a new API call
- Search is debounced 250 ms; requests use `AbortController` to prevent races

**Chunk 2 (lines ~2040–2782): visuals + themes + service worker**
- `codeToCondition(code, isDay)` — maps WMO weather codes to condition keys
- `DESCRIPTIONS`, `VISUALS`, `EMOJIS` — lookup tables for text, Lottie animations, gradients, and emoji
- Lottie player (`@lottiefiles/lottie-player`) is lazy-loaded from unpkg only when needed
- Five themes: Default, Game of Thrones, Studio Ghibli, African Savanna, Coming Soon — each overrides descriptions and/or visuals; theme is persisted in localStorage
- Service worker is registered here with skip-waiting for aggressive cache refresh

### APIs used

Both are free with no authentication:
- **Geocoding:** `https://geocoding-api.open-meteo.com/v1/search`
- **Forecast:** `https://api.open-meteo.com/v1/forecast`

### State & persistence

| Key | Storage | Purpose |
|---|---|---|
| `STORAGE_KEY` | localStorage | Last searched city (serialised place object) |
| `UNIT_KEY` | localStorage | Temperature unit preference (`celsius`/`fahrenheit`) |
| `lastData` | in-memory | Cached API response for unit-toggle re-renders |
| `lastPlace` | in-memory | Cached place object for current location |

### DOM & rendering

All DOM nodes are cached in an `el` object at startup. Forecast lists are built as HTML strings and set via `innerHTML`. Card transitions use `swapCard()` which animates opacity + transform. The `el.status` aria-live region provides screen-reader announcements.
