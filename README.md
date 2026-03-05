# WebSpatial Voice Sample

A WebSpatial app that records your voice, transcribes it with ElevenLabs Speech-to-Text, and displays the history in a second spatial panel — built with React + Vite + [@webspatial/react-sdk](https://github.com/webspatial/webspatial-sdk).

---

## Video

<!-- Add demo video here -->

---

## What it does

| Mode | Behavior |
|---|---|
| **PICO emulator / PICO OS 6** | Two floating transparent panels: mic button on the left, transcript history on the right |
| **Desktop browser** | Same UI on a single dark page, side-by-side |

- **Circular mic button** — rainbow fog glow while recording, grey fog while idle
- **ElevenLabs Scribe v2** STT — transcribes each recording on release
- **Transcript history** — auto-scrolling panel, persisted across window opens via `localStorage`

---

## Prerequisites

- [Node.js](https://nodejs.org) 18+
- An [ElevenLabs](https://elevenlabs.io) account (free tier works)
- For XR: the [PICO emulator](https://developer.picovr.com/docs/en/emulator) (Android-based) + ADB

---

## 1. Get your ElevenLabs API key

1. Sign up or log in at [elevenlabs.io](https://elevenlabs.io)
2. Click your profile icon (bottom-left) → **Profile**
3. Scroll to **API Keys** → **Create API key**
4. Copy the key — it starts with `sk_`

---

## 2. Install & configure

```bash
git clone git@github.com:nigelhartman/webspatial_voice_sample.git
cd webspatial_voice_sample

npm install

# Copy the example env file and add your key
cp .env.example .env
```

Open `.env` and replace the placeholder:

```
VITE_ELEVENLABS_API_KEY=sk_your_actual_key_here
```

---

## 3. Run in the desktop browser

```bash
npm run dev
```

Open [http://localhost:5173](http://localhost:5173) in your browser. You'll see the mic panel and transcript history side-by-side on a dark background.

---

## 4. Run in the PICO emulator

### 4a. Start the dev server (expose on all IPs)

The Vite config already sets `server.host: true`, so just run:

```bash
npm run dev
```

Note the port Vite reports — usually `5173`.

### 4b. Set up ADB port forwarding

Run this **every time** you (re)start the emulator, replacing `5173` with the actual port:

```bash
adb reverse tcp:5173 tcp:5173
```

This tunnels the emulator's requests back to your machine's dev server. Without it the emulator can't reach localhost.

### 4c. Open in PICO Browser

1. Launch the PICO emulator
2. Open the **PICO Browser**
3. Navigate to:
   ```
   http://10.0.2.2:5173/
   ```
   (`10.0.2.2` is the host machine's loopback address inside the Android emulator)
4. Tap **"Run as a standalone app"** in the address bar

The app now runs as a WebSpatial standalone app with two transparent floating panels.

### PWA requirements (if the "Run as standalone app" button is missing)

The button only appears when the page passes PWA installability checks. Verify:

- `public/manifest.json` exists and declares at least a 192×192 icon
- Icon PNG files are actually present in `public/`
- `index.html` contains `<link rel="manifest" href="/manifest.json" />`
- ADB port forwarding is active (step 4b)

---

## 5. Build for production

```bash
npm run build
```

Output goes to `dist/`. Serve it with any static host.

---

## Project structure

```
src/
  main.tsx          — entry point; detects XR mode, routes /history
  xrMode.ts         — detects standalone (XR) vs desktop browser
  VoicePage.tsx     — main panel: title + mic button + inline history (desktop)
  HistoryPage.tsx   — /history route: second WebSpatial scene (XR only)
  index.css         — spatial styles, fog animations, layout
public/
  manifest.json     — PWA manifest (required for PICO installability)
  icon-*.png        — PWA icons
.env.example        — copy to .env and add your API key
```

---

## How XR mode works

`xrMode.ts` detects `(display-mode: standalone)` — true when running as an installed PWA in the PICO emulator, false in a regular browser tab. In XR mode:

- `html.is-spatial` is added to the document → transparent WebSpatial window
- A second scene opens at `/history` via `window.open` + `initScene`
- Transcripts sync to the history panel via `BroadcastChannel` + `localStorage`

---

## Tech stack

| | |
|---|---|
| Framework | React 18 + Vite |
| XR SDK | `@webspatial/react-sdk` + `@webspatial/core-sdk` |
| STT | ElevenLabs Scribe v2 (`/v1/speech-to-text`) |
| Styling | Tailwind CSS + custom CSS animations |
