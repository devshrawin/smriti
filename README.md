<div align="center">

# 🪔 स्मृति · Smriti

**Voice memory for the person who keeps forgetting where they put their keys.**

Free forever. No account. No server. No app store. Just talk to your phone.

[![Live demo](https://img.shields.io/badge/demo-GitHub%20Pages-2F6F62?style=for-the-badge)](https://devshrawin.github.io/smriti/)
![Cost](https://img.shields.io/badge/cost-%E2%82%B90%20forever-C97A2B?style=for-the-badge)
![Stack](https://img.shields.io/badge/stack-single%20HTML%20file-2B2118?style=for-the-badge)
![Languages](https://img.shields.io/badge/भाषा-हिंदी%20%2F%20English-2F6F62?style=for-the-badge)

</div>

---

## The problem

Mom forgets where she keeps things. Keys, cash, documents — gone, and no way to ask anyone but herself an hour later. She's not technical. She doesn't want an "app" with menus. She *does* know how to talk into her phone and use WhatsApp.

## The fix

Two big buttons. That's the whole interface.

| | |
|---|---|
| 🎙️ **कुछ याद रखें** / *Remember something* | Tap → speak → "मैंने चाबियां रसोई की दराज़ में रखीं" → saved with today's date. |
| 🔍 **कुछ खोजें** / *Find something* | Tap → ask → "चाबियां कहाँ हैं" → phone **speaks back**: *"मुझे यह याद है: चाबियां रसोई की दराज़ में हैं। मुझे यह 18 मई 2026 से याद है।"* |

Home screen also shows the last 5 memories as tap-to-replay cards, plus a full searchable list. Switch the whole UI between Hindi and English with one tap — the toggle in the top corner — and it remembers the choice.

## Why this and not [other thing]

| Option | Verdict |
|---|---|
| **This PWA** | Zero servers, zero cost, zero maintenance. Works today, forever, for free. ✅ |
| Native Android app | Same OS speech engine under the hood — no functional gain, way more build/signing/distribution effort for nothing. ❌ |
| WhatsApp bot | Best possible UX (she never leaves an app she already knows) — but needs a real hosted backend (webhook + WhatsApp Business/Twilio API + speech-to-text call per message). Something to keep alive, or she's stuck the day it breaks. Good **phase 2**, not phase 1. ⏳ |

## Architecture

Deliberately boring. Boring means it never goes down.

```
Phone (Chrome, Android)
  └─ Smriti — installed to home screen as a PWA
       ├─ UI      → single index.html, inline CSS/JS
       ├─ Hear    → Web Speech API  (SpeechRecognition, hi-IN / en-IN)
       ├─ Speak   → Web Speech API  (SpeechSynthesis,   hi-IN / en-IN)
       ├─ Store   → localStorage, on-device only, JSON array
       ├─ Search  → local keyword-overlap scorer, no network, no AI
       └─ Offline → sw.js caches the shell for instant load
```

No backend. No database. No API keys. No bill, ever. The trade-off, stated plainly: memories live on *that one phone* — clearing browser data or switching phones wipes them. Backup/export is on the roadmap, not built yet.

- **Hindi speech recognition actually works**: `webkitSpeechRecognition` + `lang="hi-IN"` runs on the same Google cloud speech engine as Google Assistant's Hindi mode. No API key, free, just needs internet for that one moment of listening.
- **Search isn't AI** — it's keyword overlap against a stopword-filtered query. That's plenty for "where are the X" style questions and costs nothing to run.

## Requirements & honest limits

- Built and tested for **Chrome on Android**.
- Needs mic permission once.
- Data is per-device, per-browser — no cloud sync (yet).
- No voice support in the browser → both screens fall back to a plain text box.
- Install it properly: Chrome menu → **"Add to Home screen"** so it opens full-screen like a real app, not a browser tab.

## Design

Built for eyes and hands that don't want to fight a UI:
- Warm paper background, teal = remember, amber = find — two colors, two meanings, always.
- 26px+ buttons, 19–22px text, Hind font for crisp Devanagari.
- Home → one tap → mic → done. No settings screen to get lost in.

## Ship it: GitHub Pages in 2 minutes

This repo is already static — no build step. To get a public HTTPS link:

1. Push this repo to GitHub (already done if you're reading this from `github.com/devshrawin/smriti`).
2. On GitHub: **Settings → Pages**.
3. Under **Build and deployment → Source**, choose **Deploy from a branch**.
4. Branch: **`main`**, folder: **`/ (root)`** → **Save**.
5. Wait ~1 minute, then your app is live at:
   ```
   https://devshrawin.github.io/smriti/
   ```
6. Open that link on the phone → Chrome menu → **Add to Home screen**. Done — real app icon, works offline-shell, zero hosting cost.

Every `git push` to `main` after this redeploys automatically — no extra steps.

## Customizing

| Want to change... | Where |
|---|---|
| Font size | `.big-btn` (26px) / body text (19–22px) in `<style>` |
| Colors | Once, under `:root` — `--teal`, `--amber`, etc. |
| Search accuracy | `stopwords` arrays inside `I18N.hi` / `I18N.en` in `<script>` |
| App icon | Regenerate `icons/icon-192.png` / `icons/icon-512.png` |
| Add a language | Add a new key to the `I18N` object with the same fields as `hi`/`en`, add a toggle button |

## Roadmap

- [ ] **WhatsApp bridge** — voice-note a fixed contact instead of opening an app. Needs a small hosted webhook (Cloudflare Workers free tier) + WhatsApp Business/Twilio API + Whisper for transcription. Real infra — phase 2, only if zero-app-install becomes the priority.
- [ ] **Export/backup** — one button to export memories as text, so they survive a phone switch.
- [ ] **Synonym map** — "पैसे" / "नकद" / "रुपये" all mean "cash"; catch more phrasings without AI.
- [ ] **Auto-listen** — start the mic the instant a screen opens, save one tap.
- [ ] **Gentle nudges** — a notification asking "just put something down?" (needs PWA install + notification permission).

## Files

```
smriti/
├── index.html      # the entire app — UI, styles, logic, i18n
├── manifest.json   # PWA metadata (name, icons, colors)
├── sw.js           # service worker — offline shell caching
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
└── README.md
```

No build step, no dependencies to install. Open `index.html` in a browser, or host the folder anywhere static.

---

<div align="center">
<sub>Built because forgetting where you put your keys shouldn't need a subscription.</sub>
</div>
