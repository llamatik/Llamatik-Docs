---
title: "Stable Diffusion"
weight: 40
---

# Stable Diffusion — `/v1/stablediffusion`

The Stable Diffusion endpoints wrap `StableDiffusionBridge` for text-to-image, image-to-image, and text-to-video generation.

Images are exchanged as **Base64-encoded RGBA bytes** (`width × height × 4` bytes).
Video frames use a **packed binary format** described below.

## Initialize

### `POST /v1/stablediffusion/init`

Load a model. For standard image models only `modelPath` is required. Video models (Wan 2.1/2.2) additionally require `vaePath` and typically `t5xxlPath`.

**Standard image model:**
```json
{
  "modelPath": "/models/dreamshaper.safetensors",
  "threads": 4
}
```

**Video model (Wan 2.1 text-to-video):**
```json
{
  "modelPath": "/models/wan2.1-t2v-1.3b.gguf",
  "threads": -1,
  "vaePath": "/models/wan_2.1_vae.safetensors",
  "t5xxlPath": "/models/umt5-xxl-encoder-Q8_0.gguf"
}
```

`threads`: number of CPU threads. `-1` uses all available threads.

| Parameter | Required | Description |
|---|---|---|
| `modelPath` | ✅ | Path to the main model file |
| `threads` | | CPU thread count (default: `-1`) |
| `vaePath` | | VAE companion — required for Wan/LTX video models |
| `t5xxlPath` | | T5-XXL text encoder — typically required for video models |
| `clipVisionPath` | | CLIP vision encoder — for image-to-video variants |

**Response:**
```json
{ "ok": true }
```

All paths must resolve inside `MODELS_DIR` if that variable is configured.

## Text-to-image

### `POST /v1/stablediffusion/txt2img`

Generate an image from a text prompt.

**Request:**
```json
{
  "prompt": "A cyberpunk llama in neon Tokyo",
  "negativePrompt": "blurry, low quality",
  "width": 512,
  "height": 512,
  "steps": 20,
  "cfgScale": 7.0,
  "seed": 42
}
```

All fields except `prompt` are optional.

**Response:**
```json
{ "imageBase64": "<base64-encoded RGBA bytes>" }
```

Decode the string with `Base64.getDecoder().decode(imageBase64)` to get `width × height × 4` raw RGBA bytes.

Returns `500` if the model returns an empty byte array.

## Image-to-image

### `POST /v1/stablediffusion/img2img`

Transform an existing image guided by a text prompt.

**Request:**
```json
{
  "initImageBase64": "<base64-encoded RGBA source image>",
  "initImageW": 512,
  "initImageH": 512,
  "prompt": "The same scene as a watercolor painting",
  "negativePrompt": "low quality, blurry",
  "width": 512,
  "height": 512,
  "steps": 20,
  "cfgScale": 7.0,
  "strength": 0.75,
  "seed": 42
}
```

`strength` controls source preservation: `0.0` keeps the image almost unchanged, `1.0` ignores it entirely. Values around `0.6–0.8` work well for style transfers.

Returns `400` if `initImageBase64` is not valid Base64.

## Check video support

### `GET /v1/stablediffusion/supportsVideo`

Returns whether the currently loaded model supports video generation. Call this after `init` before offering the video feature.

```json
{ "supportsVideo": true }
```

Returns `500` if no model is loaded.

## Text-to-video

### `POST /v1/stablediffusion/txt2video`

Generate a video from a text prompt. Requires a video-capable model (Wan 2.1/2.2, LTX).

**Request:**
```json
{
  "prompt": "A llama running through a neon-lit city",
  "negativePrompt": "blurry, low quality",
  "width": 512,
  "height": 512,
  "videoFrames": 16,
  "fps": 8,
  "cfgScale": 7.0,
  "seed": 42
}
```

**Response:**
```json
{ "videoBase64": "<base64-encoded packed frame buffer>" }
```

### Packed video format

After Base64-decoding `videoBase64`, the binary layout is:

```
[frameCount : Int32 LE][width : Int32 LE][height : Int32 LE][fps : Int32 LE]   ← 16 bytes
[frame 0 RGBA : width × height × 4 bytes]
[frame 1 RGBA : width × height × 4 bytes]
...
```

Parse it in Kotlin:

```kotlin
import java.nio.ByteBuffer
import java.nio.ByteOrder

val raw = Base64.getDecoder().decode(videoBase64)
val header = ByteBuffer.wrap(raw, 0, 16).order(ByteOrder.LITTLE_ENDIAN)
val frameCount = header.int
val width      = header.int
val height     = header.int
val fps        = header.int

val frameSize = width * height * 4
val frames = (0 until frameCount).map { i ->
    raw.copyOfRange(16 + i * frameSize, 16 + (i + 1) * frameSize)
}
```

See the [Video generation guide](../../../guides/video-generation/) for saving frames to the device gallery.

## Release

### `POST /v1/stablediffusion/release`

Free Stable Diffusion native resources.

```json
{ "ok": true }
```

## Parameter guidance

| Parameter | Recommended starting value | Notes |
|---|---|---|
| `width` / `height` | `512` | Larger sizes are much slower and require more memory |
| `steps` | `20` | More steps = better quality but slower |
| `cfgScale` | `7.0` | `6.0–8.0` is a typical range |
| `seed` | any fixed value | Use `-1` for random |
| `videoFrames` | `16` | More frames = longer clip but proportionally slower |
| `fps` | `8` | Playback rate; does not affect generation speed |
