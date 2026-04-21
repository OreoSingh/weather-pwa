# Aurora Weather

A mobile-first PWA weather app with a dark glassmorphism UI, animated Lottie
weather scenes, and condition-aware background gradients.

- **Data** — [Open-Meteo](https://open-meteo.com) forecast + geocoding APIs (no key)
- **Animations** — [LottieFiles](https://lottiefiles.com) CDN, swapped by WMO code
- **Offline** — service worker caches the shell; API responses are cached as a fallback
- **Install** — ships a manifest + icons so it installs as a standalone app on iOS/Android

## Run locally

Serve the folder over HTTP (required for service workers):

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

## Deploy to GitHub Pages

1. Push `main` (or this branch) to GitHub.
2. In **Settings → Pages**, set the source to the branch root.
3. Done — the site will be at `https://<user>.github.io/weather-pwa/`.

`.nojekyll` is included so GitHub Pages serves files verbatim.
