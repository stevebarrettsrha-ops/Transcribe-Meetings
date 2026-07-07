# VoiceScript Studio 🎙️

**Private, production-ready AI transcription that runs entirely in your browser.**

Upload audio/video files or record live with your microphone, and get accurate,
timestamped transcripts powered by OpenAI's Whisper models running **on your own
device** — no server, no account, no upload. Your recordings never leave your
computer.

## Features

- **Upload anything** — MP3, MP4, M4A, WAV, OGG/Opus, WebM, FLAC, AAC, MOV.
  Drop multiple files at once; they are queued and processed one by one.
- **Record live** — capture meetings, lectures or voice notes with pause/resume
  and a live level meter, then transcribe with the same engine.
- **Accurate AI engine** — Whisper (Tiny → Large-v3-Turbo) via
  [transformers.js](https://github.com/huggingface/transformers.js), with:
  - WebGPU acceleration (automatic, with CPU/WASM fallback)
  - runs in a Web Worker so the UI never freezes
  - audio pre-processing: 16 kHz mono resample, high-pass filter, loudness
    normalization
  - silence-aware chunking so long recordings are cut at natural pauses
  - hallucination/repetition filtering
- **98+ languages** with auto-detection, plus "translate to English" mode.
- **Transcript workspace** — timestamped segments, click a timestamp to jump the
  audio player, follow-along highlighting, in-place editing (autosaved), and
  full-text search.
- **Library** — every transcript (and its source audio) is stored locally in
  your browser (IndexedDB) and searchable from the sidebar.
- **Exports** — TXT, Word (**real .docx**, dependency-free), PDF, SRT subtitles,
  WebVTT, CSV, JSON, and the original audio. Optional timestamps.
- **AI assistant (optional)** — clean-up, summary, meeting minutes, action
  items, speaker labels, translation or a custom prompt, powered by Claude.
  Bring your own [Anthropic API key](https://console.anthropic.com/); it is
  stored only in your browser and used only when you run an action.
- **Resilient** — cancellable jobs, retry from stored audio after failures or
  interrupted sessions, CDN fallbacks, friendly error messages, mobile layout.

## Getting started

The whole app is a single `index.html` — there is no build step.

### Option 1: GitHub Pages (recommended)

1. In this repository go to **Settings → Pages**.
2. Under *Build and deployment*, choose **Deploy from a branch**, select the
   `main` branch and the `/ (root)` folder, then save.
3. Open `https://<your-username>.github.io/Transcribe-Meetings/`.

### Option 2: Any static host / local server

```bash
# from the repository folder
python3 -m http.server 8080
# then open http://localhost:8080
```

Netlify, Vercel, Cloudflare Pages, S3 — anything that serves static files works.

### Option 3: Open the file directly

Double-clicking `index.html` works in Chrome/Edge for most features. Serving it
over HTTP(S) is more reliable (microphone access and workers behave best in a
secure context).

## How the engine works

1. Audio is decoded in-browser and resampled to 16 kHz mono, high-pass filtered
   at 80 Hz, and loudness-normalized — the input shape Whisper is trained on.
2. Long audio is split into ~2 minute chunks, with each cut placed at the
   quietest moment nearby so words are never chopped mid-sentence. Silent
   chunks are skipped entirely.
3. Each chunk is transcribed by Whisper (30 s windows with 5 s overlap,
   timestamps enabled) inside a Web Worker, on the GPU when WebGPU is
   available.
4. Output is filtered for the classic Whisper failure modes (repetition loops /
   hallucinated text) and streamed into the transcript view live.

Model files download once from the Hugging Face CDN (~50 MB Tiny to ~600 MB
Large-v3-Turbo) and are cached by the browser afterwards.

## Browser support

| Browser | Transcription | GPU acceleration |
|---|---|---|
| Chrome / Edge 121+ | ✅ | ✅ WebGPU |
| Firefox 141+ | ✅ | ✅ WebGPU (Windows; other platforms use CPU) |
| Safari 17+ | ✅ | CPU (WASM) |

Practical limits: very long files (3 h+) need a machine with plenty of RAM,
since decoding happens in-browser. The "Maximum" model is best used with
WebGPU; on CPU choose "Balanced".

## Privacy

- Audio, transcripts and settings are stored in your browser only (IndexedDB).
- Transcription is fully local. The only network traffic is downloading the
  engine/model files from CDNs (jsDelivr/unpkg + huggingface.co) on first use.
- The optional AI assistant sends transcript text directly from your browser
  to `api.anthropic.com`, only when you explicitly run it, using your own key.
- "Delete all data" in Settings wipes everything.

## Development

Everything lives in `index.html` (styles, markup, and the app code, including
the Web Worker source for the transcription engine and a dependency-free DOCX
writer). Edit and refresh — no toolchain required.
