---
title: "Generation"
weight: 10
---

# Generation — `/v1/generation`

The generation endpoints wrap `LlamaBridge` text generation, streaming, JSON output, KV session management, speculative decoding (MTP), and model introspection.

## Initialize

### `POST /v1/generation/init`

Load a generation model. Must be called before any other generation endpoint.

**Request:**
```json
{ "modelPath": "/models/tinyllama.gguf" }
```

**Response:**
```json
{ "ok": true }
```

Returns `400` if the model fails to load (message sanitized — no filesystem paths in the response).

---

## Text generation

### `POST /v1/generation/generate`

One-shot generation — returns when the full output is ready.

```json
{ "prompt": "Explain Kotlin Multiplatform in one sentence." }
```
```json
{ "text": "..." }
```

### `POST /v1/generation/generateWithContext`

One-shot generation with a system prompt and context block.

```json
{
  "systemPrompt": "You are a concise assistant.",
  "contextBlock": "The user is building a KMP app.",
  "userPrompt": "Which Kotlin libraries should I use?"
}
```
```json
{ "text": "..." }
```

### `POST /v1/generation/generateJson`

Schema-constrained JSON generation. The output is guaranteed to parse against the supplied JSON Schema.

```json
{
  "prompt": "Extract city and country from: I live in Barcelona, Spain.",
  "jsonSchema": "{\"type\":\"object\",\"properties\":{\"city\":{\"type\":\"string\"},\"country\":{\"type\":\"string\"}}}"
}
```
```json
{ "text": "{\"city\":\"Barcelona\",\"country\":\"Spain\"}" }
```

### `POST /v1/generation/generateJsonWithContext`

Schema-constrained JSON generation with system prompt + context.

```json
{
  "systemPrompt": "Extract structured data.",
  "contextBlock": "",
  "userPrompt": "Paris, France.",
  "jsonSchema": "..."
}
```

### `POST /v1/generation/continue`

Generate from the existing KV cache without resetting it. Call this to extend a conversation after a previous `generate` or `generateContinue`.

```json
{ "prompt": "Now show a Kotlin code example." }
```
```json
{ "text": "..." }
```

---

## Streaming (SSE)

All streaming endpoints return `Content-Type: text/event-stream`.

### `POST /v1/generation/stream`

Stream a one-shot generation.

```json
{ "prompt": "Stream a short poem." }
```

### `POST /v1/generation/streamWithContext`

Stream with system prompt + context.

```json
{
  "systemPrompt": "You are concise.",
  "contextBlock": "",
  "userPrompt": "3 bullet points on coroutines."
}
```

### `POST /v1/generation/jsonStream`

Stream schema-constrained JSON.

```json
{ "prompt": "...", "jsonSchema": "..." }
```

### `POST /v1/generation/jsonStreamWithContext`

Stream schema-constrained JSON with context.

### `POST /v1/generation/continueStream`

Stream from the existing KV cache (equivalent of `generateContinueStream` in the SDK).

```json
{ "prompt": "Continue the story." }
```

---

## Generation parameters

### `POST /v1/generation/params`

Update sampling parameters. All fields are optional — omit any you do not want to change.

```json
{
  "temperature": 0.7,
  "maxTokens": 512,
  "topP": 0.95,
  "topK": 40,
  "repeatPenalty": 1.1,
  "contextLength": 4096,
  "numThreads": 4,
  "useMmap": true,
  "flashAttention": false,
  "batchSize": 512,
  "gpuLayers": 0
}
```

`gpuLayers`: `0` = CPU only, `-1` = offload all layers to GPU.

### `POST /v1/generation/cancel`

Cancel an in-progress generation.

```json
{ "ok": true }
```

### `POST /v1/generation/shutdown`

Unload the model and free native resources.

```json
{ "ok": true }
```

---

## KV session management

### `POST /v1/generation/session/reset`

Clear the KV cache while keeping the model loaded.

```json
{ "ok": true }
```

### `POST /v1/generation/session/save`

Persist the current KV state to disk.

```json
{ "path": "/models/sessions/chat1.bin" }
```
```json
{ "ok": true }
```

`path` must resolve inside `SESSIONS_DIR` if that variable is configured.

### `POST /v1/generation/session/load`

Restore a previously saved KV state from disk. After loading, use `continue` or `continueStream` to generate from the restored context.

```json
{ "path": "/models/sessions/chat1.bin" }
```
```json
{ "ok": true }
```

---

## Model metadata

### `GET /v1/generation/model/finetuneType`

Returns the `general.finetune` metadata field from the loaded GGUF.

```json
{ "finetuneType": "instruct" }
```

| Value | Meaning |
|---|---|
| `"instruct"` / `"chat"` | Instruction-tuned — use a system + user prompt format |
| `"base"` / `null` | Base model — prompt directly |

### `GET /v1/generation/model/chatTemplate`

Returns the Jinja chat template embedded in the model, or `null` if none.

```json
{ "chatTemplate": "{% for message in messages %}..." }
```

### `POST /v1/generation/model/applyChatTemplate`

Render a list of messages into a prompt string using the model's embedded template.

```json
{
  "messages": [
    { "role": "system", "content": "You are a helpful assistant." },
    { "role": "user",   "content": "What is Kotlin?" }
  ],
  "addAssistantPrefix": true
}
```
```json
{ "prompt": "<|system|>You are a helpful assistant.<|user|>What is Kotlin?<|assistant|>" }
```

---

## Multi-Token Prediction (MTP)

MTP loads a speculative decoding head from the same GGUF, enabling draft-token speculation for faster throughput on supported models (Qwen3, GLM-4, …).

### `POST /v1/generation/mtp/init`

```json
{ "modelPath": "/models/qwen3.gguf", "draftLen": 3 }
```
```json
{ "ok": true }
```

`draftLen`: maximum draft tokens per step (1–8). Call after `/v1/generation/init`.

### `POST /v1/generation/mtp/shutdown`

Release MTP resources while keeping the base model loaded.

```json
{ "ok": true }
```
