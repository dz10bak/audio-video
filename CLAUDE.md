# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-page HTML app for creating AI-generated video from audio files. Private/personal use.

## Architecture

Single `index.html` file with embedded CSS and JS (no build tools, no frameworks). Opens directly in browser.

### User Flow
1. Upload audio file (mp3/wav/ogg/aac/webm/flac, max 10MB)
2. Audio is transcribed via deAPI (Whisper Large V3)
3. User edits the transcription as a video prompt
4. Optionally: generate a first-frame image (txt2img) for img2video
5. Generate video via deAPI (LTX-Video models)
6. Poll request status until done → display and download video

### API Integration (deAPI)

- **Base URL:** `https://api.deapi.ai`
- **Auth:** Bearer token in Authorization header
- **API key is hardcoded** (private use only)

#### Endpoints used
| Action | Method | Endpoint |
|---|---|---|
| Transcribe audio file | POST | `/api/v1/client/audiofile2txt` |
| Text to image | POST | `/api/v1/client/txt2img` |
| Text to video | POST | `/api/v1/client/txt2video` |
| Image to video | POST | `/api/v1/client/img2video` |
| Check job status | GET | `/api/v1/client/request-status/{request_id}` |
| Check balance | GET | `/api/v1/client/balance` |

#### Available Models
- **Video:** `Ltxv_13B_0_9_8_Distilled_FP8` (LTX 0.9.8, 256-768px, 30-120 frames, 30fps), `Ltx2_19B_Dist_FP8` (LTX-2, 512-1024px, 49-241 frames, 24fps)
- **Image:** `Flux1schnell`, `ZImageTurbo_INT8`, `Flux_2_Klein_4B_BF16`
- **Audio transcription:** `WhisperLargeV3`

#### Result delivery
All generation endpoints return `{ data: { request_id } }`. Poll `GET /api/v1/client/request-status/{request_id}` for status (`pending` → `processing` → `done`/`error`). On `done`, response includes `result_url`.

## Development

Open `index.html` in a browser. No server needed (API calls go directly to deAPI with CORS allowed).
