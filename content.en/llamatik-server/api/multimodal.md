---
title: "Multimodal"
weight: 50
---

# Multimodal — `/v1/multimodal`

The multimodal endpoints wrap `MultimodalBridge` for vision-language model (VLM) inference. Image analysis is returned as a streaming SSE response.

## Initialize

### `POST /v1/multimodal/init`

Load a VLM and its mmproj companion. Must be called before `analyzeStream`.

**Request:**
```json
{
  "modelPath": "/models/SmolVLM-256M-Instruct-Q8_0.gguf",
  "mmprojPath": "/models/mmproj-SmolVLM-256M-f16.gguf"
}
```

**Response:**
```json
{ "ok": true }
```

Both paths must resolve inside `MODELS_DIR` if that variable is configured.

## Analyze image (streaming)

### `POST /v1/multimodal/analyzeStream`

Analyze an image with a text prompt. Returns `Content-Type: text/event-stream`.

**Request:**
```json
{
  "imageBase64": "<base64-encoded JPEG, PNG, or BMP bytes>",
  "prompt": "Describe what you see in this image."
}
```

`imageBase64` is Base64-encoded raw image bytes (JPEG, PNG, or BMP). Returns `400` if the value is not valid Base64.

**SSE response:**
```
data: {"event":"delta","text":"A llama "}
data: {"event":"delta","text":"standing in a sunlit field."}
data: {"event":"done"}
```

On error:
```
data: {"event":"error","message":"VLM streaming failed"}
```

### Example — encode an image in Kotlin

```kotlin
val imageBytes = File("/path/to/photo.jpg").readBytes()
val imageBase64 = Base64.getEncoder().encodeToString(imageBytes)
```

### Example — consume the stream with `curl`

```bash
curl -N -X POST http://localhost:8080/v1/multimodal/analyzeStream \
  -H "Content-Type: application/json" \
  -d "{\"imageBase64\":\"$(base64 -i photo.jpg)\",\"prompt\":\"What is in this image?\"}"
```

## Cancel

### `POST /v1/multimodal/cancel`

Cancel an in-progress analysis.

```json
{ "ok": true }
```

## Release

### `POST /v1/multimodal/release`

Free VLM native resources.

```json
{ "ok": true }
```

## Notes

- The `mmprojPath` companion file must match the main model — they are trained together.
- Image encoding: the server does not resize or reformat images before passing them to the native bridge. Send the image at a resolution your model and hardware can handle.
- For large images, encode to JPEG first to reduce request size.
