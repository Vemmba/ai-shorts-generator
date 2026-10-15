# 🎬 AI Shorts Generator

> **One topic in. One finished, captioned vertical video out.** Fully automated for YouTube Shorts, TikTok & Instagram Reels.

<p align="center">
  <img alt="MIT License" src="https://img.shields.io/badge/license-MIT-blue.svg">
  <img alt="Python 3.10+" src="https://img.shields.io/badge/python-3.10%2B-blue.svg">
  <img alt="Platform" src="https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg">
</p>

<p align="center">
  Maintained by <a href="https://github.com/Vemmba"><strong>Vemmba</strong></a> · <a href="https://github.com/Vemmba">github.com/Vemmba</a>
</p>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [How It Works](#-how-it-works)
- [Quickstart](#-quickstart)
- [Usage](#-usage)
- [Footage Modes](#-footage-modes)
- [Configuration](#%EF%B8%8F-configuration)
- [Project Layout](#%EF%B8%8F-project-layout)
- [Cost per Video](#-cost-per-video)
- [FAQ](#-faq)
- [License](#-license)

## 🧭 Overview

**AI Shorts Generator** turns a single topic (or an auto-picked one from your backlog) into a ready-to-post `1080×1920` MP4 — script, visuals, AI voiceover, karaoke captions, music bed, watermark, and per-platform titles/descriptions/hashtags included. A local review dashboard lets you preview and finalize everything before publishing.

Two visual modes, mixable per video:

- 🎨 **Generate with AI** — cinematic Flux images + Wan motion clips via [fal.ai](https://fal.ai)
- 🎞️ **Clip library** — free Pexels/Pixabay stock plus a reusable semantic library (**match-then-generate**: make a shot once, reuse it forever)

## ✨ Features

- **Topic → finished video** end to end, in one command
- **Reusable clip library** with semantic matching — no paying twice for similar shots
- **AI voiceover** (OpenAI `gpt-4o-mini-tts`), tunable delivery
- **Word-by-word karaoke captions** aligned with `faster-whisper`
- **Music bed** with automatic ducking under the voice
- **Watermark & branded outro** (optional)
- **Per-platform metadata** — YouTube, TikTok, X, hashtags
- **Batch mode** — a week of content from a topic backlog
- **Local Flask dashboard** — preview, swap clips, pick music, copy captions, download
- **Prompt presets** — swap the writer's entire voice with one `.env` setting
- **Graceful degradation** — missing key or GPU? It falls back, not crashes

## 🧠 How It Works

```
topic ─▶ script (OpenAI, structured JSON)
      ─▶ visuals:  match clip library ──hit──▶ reuse clip
                                     └─miss─▶ generate (Flux image ─▶ Wan motion) ─▶ add to library
      ─▶ voiceover (gpt-4o-mini-tts)
      ─▶ captions (faster-whisper, word-level)
      ─▶ assemble (ffmpeg): Ken Burns / motion + music + watermark + outro
      ─▶ output/{date}-{slug}/final.mp4  +  per-platform titles/descriptions/hashtags
```

Output lands in `output/{date}-{slug}/final.mp4` (1080×1920) alongside `script.json` with all platform copy.

## 🚀 Quickstart

**Prerequisites:** Python 3.10+ and [FFmpeg](https://ffmpeg.org/download.html) on your `PATH`.

```bash
# 1. Clone
git clone https://github.com/Vemmba/ai-shorts-generator.git
cd ai-shorts-generator

# 2. Environment
python -m venv .venv
# Windows:      .venv\Scripts\activate
# macOS/Linux:  source .venv/bin/activate

# 3. Dependencies
pip install -r requirements.txt
python -m playwright install chromium

# 4. Configure
cp .env.example .env        # then add your API keys

# 5. One-time setup (Chromium + overlays)
python run.py setup
```

Add at least an `OPENAI_API_KEY` to `.env`. Add `FAL_KEY` for AI visuals and/or a free `PEXELS_API_KEY` for stock footage. No keys at all? It still runs an offline smoke-test mode.

```bash
python run.py make "why your mornings decide the rest of your day"
# → output/2026-06-24-why-your-mornings.../final.mp4
```

## 🎬 Usage

```bash
python run.py make ["topic"]     # one video (auto-picks a topic if omitted)
python run.py batch 7            # a week's worth from data/topics.txt
python run.py topics import      # load the backlog from data/topics.txt
python run.py fetch-library 20   # pull FREE Pexels/Pixabay stock
python run.py gen-library 10     # AI-generate clips from library/inventory.csv (fal)
python run.py library-stats      # library counts / usage
python run.py list               # list generated videos
python run.py serve              # review dashboard at http://127.0.0.1:5000
```

On Windows: double-click **`start.bat`** to launch the dashboard, **`stop.bat`** to shut it down.

### The Review Dashboard

`python run.py serve` opens a local web app to:

- write / approve / regenerate a script with your notes
- preview and **swap the library clip each scene will use**
- fetch stock or AI-generate new clips
- fit a music track and adjust its volume
- copy per-platform captions + hashtags and download the MP4

## 🎨 Footage Modes

| Mode | How | Cost | Set in `.env` |
|---|---|---|---|
| **Free Ken Burns** | AI/stock still + zoom/pan motion | free motion | `ANIMATE=none` (default) |
| **Free stock footage** | Real clips from Pexels/Pixabay | free | `PEXELS_API_KEY=...` |
| **AI images** | Flux stills via fal.ai | ~$0.025 / image | `IMAGE_BACKEND=fal` |
| **AI motion** | Wan image-to-video via fal.ai | ~$0.15 / clip | `ANIMATE=all` |
| **Local images** | SDXL-Turbo on your GPU | free | `IMAGE_BACKEND=local` (+ `requirements-gpu.txt`) |

The **clip library** catalogs every clip with embeddings, so similar scenes **reuse existing clips** instead of paying again (tune `MATCH_THRESHOLD`).

## 💧 Watermark & Outro

- **Watermark** (on by default): drop a square `assets/logo/logo.png`; add `BRAND_DISCLAIMER=` for a footer line. `WATERMARK=false` to disable.
- **Outro / end card** (off by default): `OUTRO=true` with `BRAND_NAME`, `BRAND_URL`, `BRAND_TAGLINE` + logo for a 3-second branded end card with spoken CTA.

## ⚙️ Configuration

Everything is environment-driven — see [`.env.example`](.env.example) for the full list. Highlights:

| Variable | Default | What it does |
|---|---|---|
| `OPENAI_API_KEY` | – | Script, voiceover, caption alignment, embeddings |
| `FAL_KEY` | – | AI images (Flux) + AI motion (Wan) |
| `PEXELS_API_KEY` / `PIXABAY_API_KEY` | – | Free stock footage |
| `IMAGE_BACKEND` | `fal` | `fal` (Flux) or `local` (SDXL on GPU) |
| `ANIMATE` | `none` | `none` (Ken Burns), `all`, or `N` |
| `CONTENT_NICHE` | – | Focus the writer (e.g. `personal finance`) |
| `SCRIPT_PROMPT` | – | Custom writer preset from `prompts/presets/` |
| `TTS_VOICE` | `onyx` | OpenAI TTS voice |
| `WHISPER_DEVICE` | `cpu` | `cpu` or `cuda` |
| `WATERMARK` / `OUTRO` | `true` / `false` | Toggle overlays |
| `TARGET_SECONDS` | `20` | Spoken length target |

Deeper defaults (palette, timings, models, caption styling) live in [`pipeline/config.py`](pipeline/config.py).

### ✍️ Custom Script Presets

The writer's persona lives in [`prompts/scriptgen_system.md`](prompts/scriptgen_system.md). Drop a Markdown file in [`prompts/presets/`](prompts/presets/) and select it via `SCRIPT_PROMPT` (filename without `.md`). `CONTENT_NICHE` layers a subject focus on top.

## 🗂️ Project Layout

```
run.py              CLI orchestrator
pipeline/           config, models, scriptgen, images, animate, stock,
                    library, match, voice, captions, assemble, cards, outro
dashboard/          Flask review UI (app.py + templates)
templates/          HTML/CSS for watermark + outro overlays
prompts/            scriptwriter system prompt + presets/
assets/             fonts, logo, music, generated brand overlays
library/            clip catalog (index.db) + clips/ + images/
data/topics.txt     topic backlog
output/             generated videos
```

## 💰 Cost per Video (rough)

| Setup | Approx |
|---|---|
| Script + voice + captions (OpenAI) | **~$0.02** |
| + free stock or local SDXL stills | ~$0.02 |
| + Flux AI images (5 scenes) | ~$0.15 |
| + Wan AI motion (5 scenes) | ~$0.75 |

Start cheap (`ANIMATE=none`, stock footage), turn dials up where it matters.

## ❓ FAQ

**Do I need a GPU?** No — use `IMAGE_BACKEND=fal` and `WHISPER_DEVICE=cpu`.

**Can I run it 100% free?** Mostly — OpenAI text/TTS is cents per video; stock + Ken Burns are free; AI images/motion are opt-in.

**Is auto-posting included?** No — review and post manually via the dashboard.

**What niches?** Any. Set `CONTENT_NICHE` or just pass topics.

## 🤝 Contributing

Issues and PRs are welcome at [github.com/Vemmba](https://github.com/Vemmba). Auto-posting backends, new image/i2v models, extra stock sources, and caption styles are all great starting points.

## 📄 License

[MIT](LICENSE) © [Vemmba](https://github.com/Vemmba). Based on the original work by AbdullahNaveed (MIT). Bundled fonts (Inter, Sora) are under the SIL Open Font License — see [`assets/fonts/README.md`](assets/fonts/README.md). You are responsible for the licensing of any stock footage, music, or AI-generated media you produce or redistribute.

---

<sub>Keywords: AI video generator · faceless video generator · short-form video automation · YouTube Shorts generator · TikTok video maker · Instagram Reels automation · text-to-video · AI voiceover · auto captions · fal.ai · Flux · Wan · Pexels stock footage · Python content pipeline.</sub>
