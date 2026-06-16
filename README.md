# J8NIX — Smart Operations OS, Only Smarter

A single-file, dependency-free **Smart Operations OS** dashboard for running a studio's day:
calendar, today's tasks, active projects, a daily-progress ring, live status, and a
voice-ready assistant chat — all in one clean, light, responsive web app.

**Live:** https://3shamrocksstudio.github.io/j8nix/

> Built by [3Shamrocks Studio](https://github.com/3ShamrocksStudio). Brand accent `#f2c194` on a `#ccd8e5` canvas.

## Features

- **🔥 Prompt Forge** — an AI command composer built right in. Type a request in plain language,
  pick a target (Claude / Codex / Generic) and an optional project context, and Prompt Forge
  converts it into a tight, token-efficient, well-engineered prompt — clear role, concise context,
  explicit constraints, crisp output spec. Shows an estimated token count, a **Tighten further**
  control (Standard → Tight → Tightest), and one-click **copy to clipboard**. Runs **fully offline**
  — the conversion engine is a local heuristic/template engine, no API required.
- **Editable tables** — add / edit (inline, click any cell) / delete tasks and projects; click status pills to cycle them.
- **Daily progress ring** — live SVG ring + `4/9` fraction that recomputes as tasks complete.
- **Voice — "she has a voice"**
  - **Voice input** — dictate straight into Prompt Forge with the mic button, and the
    **“Hey J8NIX”** wake word can open Prompt Forge seeded with your spoken request
    (Web Speech `SpeechRecognition`). Wake-word matching is tolerant — "Hey J8NIX",
    "Hey Jay Eight Nix", "Hey Jenix" all trigger.
  - **Voice output** — J8NIX speaks confirmations and can **read back** a generated prompt
    (`SpeechSynthesis`). The brand is always pronounced **"Jay Eight Nix"** (the synthesizer
    is fed the phonetic form, since browsers don't support SSML). She auto-picks the warmest,
    most natural system voice available (neural / Enhanced / Premium) and speaks slightly
    slower with a natural pitch; you can choose any installed voice in Settings.
  - **Premium neural voice (optional)** — for a genuinely human, warm voice, paste your own
    **ElevenLabs** API key in Settings; J8NIX then speaks through it (falling back to the
    system voice on failure). See the honest voice-quality note below.
  - Record voice memos (MediaRecorder) with in-chat playback.
- **Calm status color system** — psychologically-meaningful, distinct statuses across pills,
  row stripes, and the progress ring: To-do = slate grey, In Progress / Active = blue
  (momentum), Blocked = orange, At Risk = red, On Hold = amber, Done = green (completion).
  Brand gold/navy stay as accents.
- **Active / Done project tabs** — completing a project (status → Done) **auto-files** it under
  the **Done** tab; one click **restores** it to Active. Persisted in `localStorage`.
- **Chat** — assistant chat with history, file **drag-and-drop** upload, and notifications.
- **Quick Actions** — Prompt Forge, new project / task / reminder, project audit, record memo.
- **Drive mode** — geolocation speed detection flips the UI into a voice-first drive indicator.
- **Persistence** — tasks, projects, chat and settings saved to `localStorage`.
- **Onboarding + Settings** modal; responsive mobile layout with a bottom action bar.

## Prompt Forge: offline vs. live AI dispatch

Prompt Forge has two paths:

1. **Generate + copy (default, offline).** The local engine builds the optimized prompt with no
   network call and no API key. Copy it into Claude, Codex, or any tool. This is the recommended
   path and always works.
2. **Live "Refine with AI" (optional, advanced).** A pure front-end app **cannot securely call an
   LLM API** — any key shipped in the page is visible to everyone. So this path is **off unless you
   paste your *own* API key** into **Settings → Optional: live AI refine**. The key is stored only in
   this browser's `localStorage` and is sent directly to the provider from your browser. It is gated
   behind a clear in-app warning and the button is inert/disabled with no key. **No key is ever
   hardcoded.**

   > ⚠️ A client-side key is insecure — it's readable by any script on the page. Never use a
   > production or shared key here.

### Recommended production path — backend proxy

For a real deployment, **do not** put the key in the browser. Stand up a small backend proxy that
holds the key server-side and forwards requests:

```
Browser (no key)  ──►  Your proxy (holds ANTHROPIC_API_KEY / OPENAI_API_KEY)  ──►  LLM API
```

Minimal example (Node/Express, Anthropic):

```js
// server.js — key lives in the environment, never in the client
import express from "express";
const app = express();
app.use(express.json());

app.post("/api/refine", async (req, res) => {
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
      system: "You are a prompt engineer. Return only the improved prompt.",
      messages: [{ role: "user", content: req.body.prompt }],
    }),
  });
  res.status(r.status).json(await r.json());
});

app.listen(8787);
```

Then point the front end at `/api/refine` (same-origin, no key in the client, add auth/rate-limiting
as needed). The in-browser BYO-key path exists only for local experimentation.

## Voice quality — honest note

The built-in voice uses the browser's **Web Speech `SpeechSynthesis`** API. We do everything
possible client-side to make it warm: J8NIX scores the device's installed voices and picks the
most natural one (neural / "Enhanced" / "Premium" — e.g. Samantha (Enhanced), Ava, Google neural,
Microsoft Aria/Jenny), pronounces the brand phonetically as "Jay Eight Nix", and speaks slightly
slower with a natural pitch for a less robotic cadence.

**But browser TTS is inherently limited** — even the best system voice is not truly human. A
genuinely warm, human voice requires a **cloud neural TTS** (ElevenLabs, Google, Azure). Two paths:

1. **Optional in-app hook (now):** paste your own **ElevenLabs** API key in
   **Settings → premium neural voice**. J8NIX calls ElevenLabs directly and plays the audio,
   falling back to the system voice on any error. This is gated behind a clear warning: a
   client-side key is exposed to page scripts — use a limited key, never a production one.
2. **Recommended production path:** a **backend proxy** that holds the TTS key server-side
   (same shape as the Prompt Forge proxy above) and returns audio to the client. This keeps the
   key secret and is the right way to ship a human voice to real users.

`connect-src` is limited to `api.anthropic.com`, `api.openai.com`, and `api.elevenlabs.io` for
these optional, key-gated paths only.

## Tech & security

- **One standalone `index.html`** — no build step, no external JS dependencies (Google Fonts only).
- Content injected via `textContent` / `createElement` — **no `innerHTML` of user data, no `eval`**.
- Ships a **Content-Security-Policy** meta tag and is CSP-friendly. `connect-src` allows only
  `api.anthropic.com`, `api.openai.com`, and `api.elevenlabs.io` so the optional, user-key-gated
  "Refine with AI" and premium-voice paths can run; nothing else is permitted to phone home.
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
