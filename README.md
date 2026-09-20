# स्मृति (Smriti) — Voice Memory Helper

A free, installable, single-file web app (PWA) built to help someone who forgets where they put things (keys, cash, documents, etc.). No account, no server, no ongoing cost — runs entirely in the phone's browser, on-device.

Built for a Hindi-speaking, Android/Chrome, WhatsApp-familiar user who is not technical.

---

## 1. What it does

Two big buttons on the home screen:

| Button | What happens |
|---|---|
| 🎙️ **कुछ याद रखें** (Remember something) | Tap the mic, say a sentence like "मैंने चाबियां रसोई की दराज़ में रखीं" (I put the keys in the kitchen drawer). It's transcribed into a text box, she can check/edit it, then tap **सेव करें** (Save). Stored with today's date automatically. |
| 🔍 **कुछ खोजें** (Find something) | Tap the mic, ask "चाबियां कहाँ हैं" (Where are the keys). The app searches everything saved, and **speaks the answer out loud**, e.g. "मुझे यह याद है: चाबियां रसोई की दराज़ में हैं। मुझे यह 18 मई 2026 से याद है।" |

The home screen also lists the 5 most recent memories as tap-to-hear-again cards, and there's a "जो कुछ भी मुझे याद है" (Everything I remember) screen to browse/delete the full list.

## 2. Architecture

Deliberately the simplest design that satisfies "cheapest, free, always works, nobody has to maintain a server":

```
Phone (Chrome, Android)
  └─ Smriti PWA (installed to home screen)
       ├─ UI: single index.html, inline CSS/JS, Devanagari-friendly font
       ├─ Input:  Web Speech API (SpeechRecognition, hi-IN)  → text
       ├─ Output: Web Speech API (SpeechSynthesis, hi-IN)    → voice reply
       ├─ Storage: browser localStorage (JSON array, on-device only)
       ├─ Search: local keyword-overlap scorer (no network, no AI call)
       └─ Offline shell: sw.js caches the static files
```

- **Speech-to-text**: Browser's built-in `SpeechRecognition` API (`webkitSpeechRecognition`), `lang=hi-IN`. This runs on Google's cloud speech engine on Android Chrome — the same engine behind Google Assistant's Hindi recognition. Free, no API key, needs internet.
- **Text-to-speech**: Browser's built-in `speechSynthesis` API, also `hi-IN`.
- **Storage**: `localStorage` — a JSON array of `{ text, timestamp, keywords }` objects. Nothing leaves the phone; there is no backend/database.
- **Search**: naive keyword-overlap scoring. Query words (minus Hindi stopwords like "है", "में", "को") are matched against each saved memory's keywords; highest-scoring memory is read aloud. Not AI — just word matching, sufficient for short "where did I put X" queries.
- **PWA shell**: `manifest.json` + `sw.js` make it installable to the home screen with its own icon, and cache the static shell for instant load (voice features still need internet).

No backend, no database, no API keys, no recurring cost. Trade-off: data lives only on this phone/browser — no cross-device sync or automatic backup (see Limits below).

### Why not native app or WhatsApp bot?

| Option | Verdict |
|---|---|
| Native Android app | Uses the same underlying OS speech APIs as the PWA — no functional gain, much more build/deploy effort (Kotlin, signing, distribution). Not worth it here. |
| WhatsApp bot | Best possible UX since she already lives in WhatsApp — no new app icon at all. But requires a real backend: a hosted webhook, WhatsApp Business/Twilio API, and a speech-to-text API call per message. That's ongoing infrastructure someone has to keep running; if it goes down, she can't find her keys. Viable as a **phase 2** once the PWA is proven, not as the first version. |
| This PWA | Zero servers, zero cost, zero maintenance surface. Ships today. |

## 3. Requirements & limits

- Works best in **Chrome on Android** (built and tested for this).
- Needs microphone permission the first time (browser will prompt).
- Data is tied to **that specific phone + browser**. Clearing browser data/cache, or switching phones, wipes it — no cloud backup built in yet.
- If voice recognition doesn't work (unsupported browser), both screens have a text box as a fallback — she can type instead of speaking.
- Use the browser menu → **"Add to Home screen"** (or the install prompt Chrome shows automatically) so it opens as a standalone app with its own icon, not a browser tab.

## 4. Design

Warm, high-contrast, large-touch-target design built for an elderly user:
- Paper-cream background, teal for "remember" actions, amber for "find" actions.
- 26px+ buttons, 19-22px body text, Hind font for clean Devanagari rendering.
- Minimal steps: home → one button → mic → done. No menus, no settings screen to get lost in.

## 5. Customizing it further

- **Font size** — search `font-size` in the `<style>` section (`.big-btn` is `26px`, body text `19-22px`).
- **Colors** — defined once under `:root` (`--teal`, `--amber`, etc.).
- **Stopwords / search accuracy** — the `STOPWORDS` set in `<script>`; add filler words she tends to use if search misses things.
- **App icon** — regenerate `icons/icon-192.png` / `icons/icon-512.png` (see below).

## 6. Ideas / next steps

- **WhatsApp bridge**: let her send a voice note to a fixed WhatsApp contact instead of opening an app. Needs Meta Cloud API (free tier, limited messages/month) or Twilio WhatsApp sandbox + a small serverless webhook (e.g. Cloudflare Workers free tier) that transcribes (Whisper API) and replies. Real infra to maintain — the natural phase 2 if the PWA proves out but zero-app-install UX becomes the priority.
- **Backup/export**: since data is localStorage-only, add an export-to-text or share-to-WhatsApp-self button so it can be periodically backed up.
- **Synonym map**: e.g. map "पैसे"/"नकद"/"रुपये" all to "cash" so search catches more phrasings — no AI needed, just a lookup table alongside `STOPWORDS`.
- **Auto-listen on screen open**: start the mic automatically when she opens "याद रखें"/"खोजें" to cut a tap.
- **Gentle reminder nudges**: a periodic notification asking "just put something down? want to save it?" — needs the PWA installed and Notification permission; more complex, optional.

## 7. Files

```
smriti/
├── index.html      # the entire app — UI, styles, logic
├── manifest.json   # PWA metadata (name, icons, colors)
├── sw.js           # service worker — offline shell caching
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
└── README.md
```

Open `index.html` directly in Chrome, or host the folder anywhere static (GitHub Pages, Netlify, etc.) — no build step needed.
