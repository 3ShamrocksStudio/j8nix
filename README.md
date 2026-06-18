# J8NIX — 3Shamrocks Studio Command Center

A single-file, dependency-free **mission-control dashboard** for the whole 3Shamrocks studio.
One place where Dave sees **every 3S project, its status, and exactly what's waiting on him** —
everything clickable and actionable.

**Live:** https://3shamrocksstudio.github.io/j8nix/

> Built by [3Shamrocks Studio](https://github.com/3ShamrocksStudio). J8NIX™ gold wordmark on a deep-navy command canvas.

## The view

The key view fits **above the fold** on desktop and stacks cleanly on mobile — scroll only inside panels where it's logical.

- **⏳ Waiting on You** (left rail, priority queue) — each item shows the project, a priority dot,
  the exact revisions/decisions expected, and a button that opens **the precise thing to review,
  test or decide**. Highest priority on top. Add or clear items as you go.
- **🗂 Projects** (grid) — every 3S project as a card with:
  - a **live status pill** — 🟢 Live / 🔵 In Progress / 🟡 Waiting on Dave / ⚪ Backlog (click to cycle);
  - the next action it needs;
  - a working **Open / Test live** button to the **real URL** (and a **Kit** button where one exists).
  - **Filter chips** (All / Live / In progress / Waiting / Backlog) with live counts.
- **💬 Message Claude** (composer) — type a command, attach optional project context, hit **Copy**.
  It formats a ready-to-paste message for Claude in chat. **Honest:** it does **not** silently send
  to a live AI — that bridge needs a backend (see below). Labeled "copy & send · live bridge: future phase".
- **Per-project detail** — click any card (or a waiting item's *Open project*) to see its
  **ship checklist**: tick steps off, add/remove steps, jump to the live URL, or seed a Claude command about it.

Everything persists to `localStorage`, so Dave's edits, statuses and checklists stick.

## Projects tracked (real, verified links)

All product links were verified to return **HTTP 200**:

| Project | Link |
| --- | --- |
| SHOMER · Forest | https://3shamrocksstudio.github.io/shomer-forest/ |
| SHOMER · Beach | https://3shamrocksstudio.github.io/shomer-beach/ |
| Milo | https://3shamrocksstudio.github.io/milo-app/ |
| Chore Wars | https://3shamrocksstudio.github.io/chore-wars/ |
| Senior Helper | https://3shamrocksstudio.github.io/senior-helper/ |
| Trashure-Hunters · Site | https://3shamrocksstudio.github.io/th-website/ |
| 3S Design System | https://3shamrocksstudio.github.io/3s-design-system/ |
| Media Network Hub (+ Kit) | https://3shamrocksstudio.github.io/media-network-hub/ · [/kit.html](https://3shamrocksstudio.github.io/media-network-hub/kit.html) |
| Gumroad product | https://markodave.gumroad.com/l/ruaqk |

Plus in-flight items that ship before they have a public URL: **Trashure-Hunters MVP**
(regenerate Anthropic key + add Firebase creds), **TH Movie** (script length ~55 vs ~90),
**Cosmix** (pick direction A or C), and **IG Network** (create 4 accounts — paused on final names).

## What's real vs. what needs a backend

- **Real now:** every status, checklist, filter, waiting item and link works and persists in the browser.
  Open/Test buttons open the real deployed products. The Message Claude composer formats and copies a
  ready-to-paste command.
- **Future phase (needs a backend):** a **live AI bridge** — Message Claude sending a command straight
  to an LLM and showing the reply — cannot be done securely from a static page (any key shipped in the
  page is public). The composer is therefore honest copy-to-paste today.

### 🔥 Prompt Forge (bonus, carried over)

Type a request in plain words; Forge converts it into a tight, token-efficient prompt (clear role,
context, constraints, output spec) with a token estimate and **Tighten further** control. Runs **fully
offline** — no API key required. An optional **Refine with AI** path is off unless you paste your *own*
key in Settings (stored only in this browser, gated behind a clear warning).

### Recommended production path — backend proxy

For a real live-AI deployment, **do not** put the key in the browser. Stand up a small proxy that holds
the key server-side and forwards requests:

```
Browser (no key)  ──►  Your proxy (holds ANTHROPIC_API_KEY)  ──►  Claude API
```

```js
// server.js — key lives in the environment, never in the client
import express from "express";
const app = express();
app.use(express.json());

app.post("/api/claude", async (req, res) => {
  const r = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: {
      "content-type": "application/json",
      "x-api-key": process.env.ANTHROPIC_API_KEY, // server-side secret
      "anthropic-version": "2023-06-01",
    },
    body: JSON.stringify({
      model: "claude-opus-4-8",
      max_tokens: 1024,
      messages: [{ role: "user", content: req.body.prompt }],
    }),
  });
  res.status(r.status).json(await r.json());
});

app.listen(8787);
```

Then point the composer at `/api/claude` (same-origin, no key in the client, add auth + rate-limiting).

## Voice (optional)

The **🔊 Brief** button reads back what's waiting on you using the browser's `SpeechSynthesis`
(brand pronounced "Jay Eight Nix"). For a genuinely human voice, paste your own **ElevenLabs** key in
Settings — gated behind a clear warning (a client-side key is exposed to page scripts; the production-safe
path is a backend proxy). `connect-src` is limited to `api.anthropic.com`, `api.openai.com`, and
`api.elevenlabs.io` for these optional, key-gated paths only.

## Tech & security

- **One standalone `index.html`** — no build step, no external JS dependencies (Google Fonts only).
- Content injected via `textContent` / `createElement` — **no `innerHTML` of user data, no `eval`**.
- Ships a strict **Content-Security-Policy** meta tag; nothing phones home except the optional key-gated paths.
- **No service worker.** GitHub Pages org projects share one origin, so a stale SW can hijack sibling
  pages — on load the page actively **unregisters service workers** and purges caches it doesn't own.
- **WCAG AA** — focus-visible rings, focus-trapped modal, ARIA labels, keyboard-operable controls,
  reduced-motion support, responsive down to 375px.

## Run locally

```bash
python3 -m http.server 8123
# then open http://localhost:8123/index.html
```

## License

© 2026 3Shamrocks Studio. All rights reserved. **J8NIX™** is a trademark of 3Shamrocks Studio.
