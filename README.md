# J8NIX — Smart Operations OS, Only Smarter

A single-file, dependency-free **Smart Operations OS** dashboard for running a studio's day:
calendar, today's tasks, active projects, a daily-progress ring, live status, and a
voice-ready assistant chat — all in one clean, light, responsive web app.

**Live:** https://3shamrocksstudio.github.io/j8nix/

> Built by [3Shamrocks Studio](https://github.com/3ShamrocksStudio). Brand accent `#f2c194` on a `#ccd8e5` canvas.

## Features

- **Editable tables** — add / edit (inline, click any cell) / delete tasks and projects; click status pills to cycle them.
- **Daily progress ring** — live SVG ring + `4/9` fraction that recomputes as tasks complete.
- **Voice**
  - Record voice memos (MediaRecorder) with in-chat playback.
  - Text-to-speech replies using a warm female voice when available.
  - Wake-word listening — say **“Hey J8NIX”** (Web Speech `SpeechRecognition`).
- **Chat** — assistant chat with history, file **drag-and-drop** upload, and notifications.
- **Quick Actions** — new project / task / reminder, project audit, record memo.
- **Drive mode** — geolocation speed detection flips the UI into a voice-first drive indicator.
- **Persistence** — tasks, projects, chat and settings saved to `localStorage`.
- **Onboarding + Settings** modal; responsive mobile layout with a bottom action bar.

## Tech & security

- **One standalone `index.html`** — no build step, no external JS dependencies (Google Fonts only).
- Content injected via `textContent` / `createElement` — **no `innerHTML` of user data, no `eval`**.
- Ships a **Content-Security-Policy** meta tag and is CSP-friendly.
- **No service worker.** GitHub Pages org projects share one origin, so a stale SW can hijack
  sibling pages (lesson from the SHOMER deploy). To stay safe, on load the page actively
  **unregisters any service workers** and purges caches it doesn't own.

## Run locally

```bash
# any static server works, e.g.
python3 -m http.server 8731
# then open http://localhost:8731/index.html
```

Voice, microphone and geolocation features require a real browser (and HTTPS or `localhost`)
and user permission; they degrade gracefully when unavailable.

## Custom domain (future)

A future custom-domain option is **`j8nix.3shamrocks.studio`**. To enable it later:
add a `CNAME` file containing that hostname, point a DNS `CNAME` record at
`3shamrocksstudio.github.io`, and set the domain under the repo's Pages settings.
(No DNS is configured yet.)

## License

© 2026 3Shamrocks Studio. All rights reserved. **J8NIX™** is a trademark of 3Shamrocks Studio.
