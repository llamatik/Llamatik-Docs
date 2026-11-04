---
title: "Whisper"
weight: 30
---

# Whisper — `/v1/whisper`

The Whisper endpoints wrap `WhisperBridge` speech-to-text. Two transcription modes are available: flat text (`transcribe`) and structured per-segment JSON (`transcribeSegments`).

## Initialize

### `POST /v1/whisper/init`

Load a Whisper GGML model. Must be called before any transcription endpoint.

**Request:**
```json
{ "modelPath": "/models/ggml-base.en.bin" }
```

**Response:**
```json
{ "ok": true }
```

## Transcribe

### `POST /v1/whisper/transcribe`

Transcribe a WAV file to a flat string.

**Request:**
```json
{
  "wavPath": "/audio/recording.wav",
  "language": "en",
  "initialPrompt": "Technical vocabulary"
}
```

`language` and `initialPrompt` are optional.

- `language` — ISO 639-1 code (e.g. `"en"`, `"de"`, `"fr"`). Omit to auto-detect.
- `initialPrompt` — prime the model with vocabulary or context to improve accuracy on domain-specific content.

**Response:**
```json
{ "text": "Kotlin Multiplatform is a great technology." }
```

Provide a 16 kHz, mono, 16-bit PCM WAV for best results.

## Segment-aware transcription

### `POST /v1/whisper/transcribeSegments`

Transcribe a WAV file and return per-segment JSON with timestamps, auto-detected language, and optional speaker-turn markers.

**Request:**
```json
{
  "wavPath": "/audio/recording.wav",
  "language": null,
  "translate": false,
  "diarize": false
}
```

All fields except `wavPath` are optional.

- `translate` — `true` to output English translations of non-English audio (Whisper's built-in translate task).
- `diarize` — `true` only with a `…-tdrz` tinydiarize model; enables `speaker_turn_next` boundaries.

**Response:**
```json
{
  "json": "{\"language\":\"en\",\"segments\":[{\"text\":\"Hello.\",\"t0\":0,\"t1\":800,\"speaker_turn_next\":false}]}"
}
```

The `json` field contains a string of JSON. Decode it to get the structure:

```json
{
  "language": "en",
  "segments": [
    { "text": "Hello.", "t0": 0, "t1": 800, "speaker_turn_next": false }
  ]
}
```

- `t0` / `t1` — segment start and end in **milliseconds**
- `speaker_turn_next` — `true` only when using a tinydiarize model and a speaker change follows

## Release

### `POST /v1/whisper/release`

Free Whisper native resources.

```json
{ "ok": true }
```

## Notes

- Reuse the initialized model across multiple transcription calls.
- `wavPath` must resolve inside `MODELS_DIR` if that variable is configured.
