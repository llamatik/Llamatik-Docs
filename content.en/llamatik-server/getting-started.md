---
title: "Getting started"
weight: 10
---

# Getting started

This page walks through installing Llamatik Server, running it, and making your first inference request.

## Requirements

- JVM 21+
- A GGUF model file (e.g. `tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf`)
- Docker (optional, for containerised deployment)

## Clone and build

```bash
git clone https://github.com/ferranpons/Llamatik-Server.git
cd Llamatik-Server
./gradlew build
```

## Run locally

```bash
./gradlew run
```

The server starts on `http://localhost:8080` by default.

## Minimal environment

```bash
export JWT_SECRET="change-me-in-production"
export JDBC_DRIVER="org.h2.Driver"
export JDBC_DATABASE_URL="jdbc:h2:mem:llamatik;DB_CLOSE_DELAY=-1"
```

`JWT_SECRET` signs user tokens. `JDBC_*` configure the user/profile database (H2 in-memory is fine for local development; use PostgreSQL for production). To create the PostgreSQL database and schema, run `./scripts/setup-db.sh` from the server repository (idempotent; options `-h`, `-p`, `-U`, `-d`). See [Configuration](../configuration/) for all variables.

## Load a model and generate text

> You can skip this step by setting `GENERATION_MODEL_PATH` so the model loads at startup — see [Configuration](../configuration/#startup-model-loading). `GET /v1/models` shows what is loaded.

**Step 1 — initialize the generation model:**

```bash
curl -s -X POST http://localhost:8080/v1/generation/init \
  -H "Content-Type: application/json" \
  -d '{"modelPath": "/models/tinyllama.gguf"}' | jq
```

```json
{ "ok": true }
```

**Step 2 — generate text:**

```bash
curl -s -X POST http://localhost:8080/v1/generation/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Explain Kotlin Multiplatform in one sentence."}' | jq
```

```json
{ "text": "Kotlin Multiplatform lets you share business logic across Android, iOS, and Desktop from a single codebase." }
```

**Step 3 — stream tokens:**

```bash
curl -N -X POST http://localhost:8080/v1/generation/stream \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Write a haiku about a llama."}'
```

```
data: {"event":"delta","text":"A "}
data: {"event":"delta","text":"llama "}
data: {"event":"delta","text":"wanders."}
data: {"event":"done"}
```

## Load a Whisper model and transcribe audio

```bash
curl -s -X POST http://localhost:8080/v1/whisper/init \
  -H "Content-Type: application/json" \
  -d '{"modelPath": "/models/ggml-base.en.bin"}' | jq

curl -s -X POST http://localhost:8080/v1/whisper/transcribe \
  -H "Content-Type: application/json" \
  -d '{"wavPath": "/audio/recording.wav", "language": "en"}' | jq
```

## Load an image model and generate an image

```bash
curl -s -X POST http://localhost:8080/v1/stablediffusion/init \
  -H "Content-Type: application/json" \
  -d '{"modelPath": "/models/dreamshaper.safetensors", "threads": 4}' | jq

curl -s -X POST http://localhost:8080/v1/stablediffusion/txt2img \
  -H "Content-Type: application/json" \
  -d '{"prompt": "A watercolor llama in the mountains", "width": 512, "height": 512}' | jq
```

The `imageBase64` field in the response is a Base64-encoded RGBA byte array.

## Next steps

- Configure path security and authentication: [Configuration](../configuration/)
- Explore all endpoints: [API Reference](../api/)
- Run in Docker or as a systemd service: [Deployment](../deployment/)
