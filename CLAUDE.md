# CLAUDE.md — Anna English

This file provides guidance for AI assistants working on this codebase.

## Project Overview

**Anna English** is a minimal Progressive Web App (PWA) that teaches English phrases over 100 days. It targets Japanese-speaking users learning English and is designed to be installed as a standalone mobile app. There are no external dependencies, no build tools, and no backend — everything runs in the browser from static files.

## Repository Structure

```
anna-english/
├── index.html      # Entire application: HTML structure, inline CSS, and inline JS
├── manifest.json   # PWA manifest (app name, icon, theme color, display mode)
├── sw.js           # Service Worker for offline caching
├── README.md       # Minimal project description
└── CLAUDE.md       # This file
```

**Note:** `icon.png` is referenced in `manifest.json` but is not present in the repository. This causes a 404 warning in the browser console but does not break the app.

## Technology Stack

| Layer | Technology |
|---|---|
| Markup | HTML5 |
| Styling | Vanilla CSS3 (inline in `index.html`) |
| Logic | Vanilla JavaScript ES6+ (inline in `index.html`) |
| Offline | Service Worker (`sw.js`) |
| PWA | Web App Manifest (`manifest.json`) |
| Audio | Web Speech API (`speechSynthesis`) |
| Storage | `localStorage` |
| Build | None — no package manager, bundler, or compiler |

## Development Workflow

There is no build step. The app runs directly from static files.

### Local Development

```bash
# Option 1: Open directly in a browser
open index.html

# Option 2: Serve via a local HTTP server (required for Service Worker)
python3 -m http.server 8080
# then visit http://localhost:8080
```

**Important:** Service Workers only register on `localhost` or over HTTPS. For offline functionality to work during development, use a local server rather than opening `index.html` as a `file://` URL.

### Deployment

Any static file host works:
- GitHub Pages
- Netlify / Vercel
- Apache / Nginx

There is no `.env` file, secrets, or server-side configuration.

## Code Conventions

All application code lives in the single `<script>` block at the bottom of `index.html`. Conventions used throughout:

- **No semicolons** in some places; the style is slightly inconsistent — do not over-correct.
- **Compact style**: minimal whitespace, short variable names, concise expressions.
- **No classes**: plain functions and module-level variables.
- **`onclick` attributes** on buttons rather than `addEventListener`.
- **Ternary initialization**: `localStorage.getItem(key) ? parseInt(...) : 0`.

## Data Structures

### `phrases` (Array of [English, Japanese] pairs)

```js
const phrases = [
  ["I'm ready!", "準備できたよ！"],
  ["Let's go!",  "いこう！"],
  // ... 20 hand-curated pairs ...
];

// Padded to exactly 100 entries with a filler phrase
while (phrases.length < 100) {
  phrases.push(["Keep going!", "そのまま続けて！"]);
}
```

Indices `0–19` contain unique phrases. Indices `20–99` are filler.

### `characters` (Decorative emoji)

```js
const characters = ["👑✨", "🩰🌸", "💃🔥", "🦄🌈", "🌟🎀", "🧚‍♀️✨"];
```

One emoji combination is chosen at random each time `showPhrase()` runs.

### `currentDay` (User state)

```js
let currentDay = localStorage.getItem("annaDay")
  ? parseInt(localStorage.getItem("annaDay")) : 0;
```

- Stored in `localStorage` under the key `"annaDay"`.
- Zero-indexed (`0–99`); displayed to users as Day 1–100.
- Persists across browser sessions and app installs.

## Core Functions

| Function | Description |
|---|---|
| `showPhrase()` | Renders the current day's phrase, day number, and a random emoji to the DOM |
| `nextDay()` | Increments `currentDay` (max 99), saves to `localStorage`, calls `showPhrase()` |
| `prevDay()` | Decrements `currentDay` (min 0), saves to `localStorage`, calls `showPhrase()` |
| `speakPhrase()` | Uses `SpeechSynthesisUtterance` with `lang="en-US"` to read the English phrase aloud |

## PWA Details

### manifest.json

- **Display mode**: `standalone` — hides browser chrome when installed.
- **Theme color**: `#ff6f61` (coral).
- **Background color**: `#ff9a9e` (light pink) — shown on the splash screen.
- **Icon**: `icon.png` at 192×192px (file missing from repo).

### sw.js (Service Worker)

Uses a **cache-first** strategy:

1. On `install`: caches `index.html` into a cache named `"anna-cache"`.
2. On `fetch`: returns the cached response if available; otherwise fetches from the network.

The service worker is registered at the bottom of `index.html`:

```js
if ("serviceWorker" in navigator) {
  navigator.serviceWorker.register("sw.js");
}
```

## UI Layout

The UI is a single centered card (`.card`) with:
- A random emoji character (`.character`)
- Day number heading (`<h2>`)
- English phrase (`.english`, bold, 24px)
- Japanese translation (`.japanese`, gray)
- Three full-width buttons: **Speak** (green), **Next Day** (coral), **Back** (blue)

Design is mobile-first with a max card width of 420px and a pink gradient background.

## Known Issues / Gaps

- **`icon.png` missing**: The manifest references `icon.png` but the file is not committed. Add a 192×192 PNG to fix the PWA installation warning.
- **Filler content**: Days 21–100 all show the same "Keep going!" phrase. Adding more unique phrases would improve the experience.
- **No completion state**: When the user reaches Day 100, there is no end screen or reset mechanism.
- **Cache versioning**: `sw.js` has no cache-busting mechanism. After updates, users may see stale content until the service worker updates.

## Language Note

The `<html>` element has `lang="ja"` (Japanese), which is appropriate since the primary UI text and navigation language is Japanese. The English phrases are read aloud via the Speech API with `lang="en-US"`.
