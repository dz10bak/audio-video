# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-page HTML app ("AI Studio") for generating AI speech from text (TTS) and creating AI video. Private/personal use. Two main tabs: "Generuj Audio" and "Tworzenie Wideo".

## Architecture

Single `index.html` file with embedded CSS and JS (no build tools, no frameworks, vanilla JS). Opens directly in browser — no server needed (API calls go directly to deAPI with CORS allowed).

### Tab 1: Generuj Audio (Text-to-Speech)
1. User types text (min 10 chars — shorter text = shorter audio)
2. Selects TTS model, language, voice, speed, format
3. Generates audio via deAPI → polls for result → plays audio
4. "Uzyj w tworzeniu wideo" button passes generated audio to Tab 2

### Tab 2: Tworzenie Wideo
1. Audio source: upload file (max 10MB) OR use generated audio from Tab 1
2. Transcribe audio via Whisper (or skip transcription)
3. Edit transcription as video prompt
4. Optionally: generate first-frame image (txt2img → img2video)
5. Generate video → poll status → display and download

### API Integration (deAPI)

- **Base URL:** `https://api.deapi.ai`
- **Auth:** Bearer token in Authorization header
- **API key is hardcoded** (private use only)

#### Endpoints used
| Action | Method | Endpoint | Content-Type |
|---|---|---|---|
| Text to speech | POST | `/api/v1/client/txt2audio` | application/json |
| Transcribe audio file | POST | `/api/v1/client/audiofile2txt` | multipart/form-data |
| Text to image | POST | `/api/v1/client/txt2img` | application/json |
| Text to video | POST | `/api/v1/client/txt2video` | multipart/form-data |
| Image to video | POST | `/api/v1/client/img2video` | multipart/form-data |
| Check job status | GET | `/api/v1/client/request-status/{request_id}` | — |
| Check balance | GET | `/api/v1/client/balance` | — |

#### Available Models
- **TTS:** `Kokoro` (fast, natural, en/es/fr/it/pt/hi, many voices, speed 0.5-2x), `Chatterbox` (20+ languages incl. Polish, single "default" voice, speed fixed 1x), `Qwen3_TTS_12Hz_1_7B_CustomVoice` (premium, 10 languages, multiple voices, speed fixed 1x)
- **Video:** `Ltx2_19B_Dist_FP8` (LTX-2: 512-1024px, 49-241 frames, 24fps, steps min 8), `Ltxv_13B_0_9_8_Distilled_FP8` (LTX 0.9.8: 256-768px, 30-120 frames, 30fps, steps min 1)
- **Image:** `Flux1schnell`, `ZImageTurbo_INT8`, `Flux_2_Klein_4B_BF16`
- **Transcription:** `WhisperLargeV3`

#### Critical API requirements
- **Video `fps` field is required** — 24 for LTX-2, 30 for LTX 0.9.8
- **Video `steps`** — LTX-2 requires minimum 8; LTX 0.9.8 accepts 1
- All generation endpoints return `{ data: { request_id } }`. Poll `GET /api/v1/client/request-status/{request_id}` for status (`pending` → `processing` → `done`/`error`). On `done`, response includes `result_url`.
- TTS `sample_rate` is always 24000

## Development

Open `index.html` in a browser. Reload to see changes. All state is in-memory (no localStorage/backend).
