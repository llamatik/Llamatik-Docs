---
title: "StableDiffusionBridge"
weight: 3
---

`StableDiffusionBridge` provides on-device text-to-image generation.
It is intended for apps that want local image generation without sending prompts or generated images to external servers.

## Platform support

Current support in the library code:

- **Android**: supported
- **iOS**: supported
- **JVM / Desktop**: supported
- **WASM**: currently not implemented

## Public API

```kotlin
expect object StableDiffusionBridge {
    fun getModelPath(modelFileName: String): String
    fun initModel(
        modelPath: String,
        threads: Int = -1,
        vaePath: String? = null,
        t5xxlPath: String? = null,
        clipVisionPath: String? = null,
    ): Boolean
    fun txt2img(
        prompt: String,
        negativePrompt: String? = null,
        width: Int = 512,
        height: Int = 512,
        steps: Int = 20,
        cfgScale: Float = 7.0f,
        seed: Long = -1L,
    ): ByteArray
    fun img2img(
        initImageRgba: ByteArray,
        initImageW: Int,
        initImageH: Int,
        prompt: String,
        negativePrompt: String? = null,
        width: Int = 512,
        height: Int = 512,
        steps: Int = 20,
        cfgScale: Float = 7.0f,
        strength: Float = 0.75f,
        seed: Long = -1L,
    ): ByteArray
    fun release()
    fun supportsVideoGeneration(): Boolean
    fun txt2video(
        prompt: String,
        negativePrompt: String? = null,
        width: Int = 512,
        height: Int = 512,
        videoFrames: Int = 16,
        fps: Int = 8,
        cfgScale: Float = 7.0f,
        seed: Long = -1L,
    ): ByteArray
}
```

## `getModelPath(modelFileName)`
Returns a usable path for the model file on the current platform.

```kotlin
val modelPath = StableDiffusionBridge.getModelPath("dreamshaper.safetensors")
```

## `initModel(modelPath, threads, vaePath, t5xxlPath, clipVisionPath)`
Loads the Stable Diffusion model and creates the native context.

```kotlin
// Standard model (SD, SDXL, …)
val ok = StableDiffusionBridge.initModel(modelPath, threads = 4)
check(ok) { "Failed to initialize Stable Diffusion model" }

// DiT video model (Wan, LTX) — requires companion files
val ok = StableDiffusionBridge.initModel(
    modelPath      = modelPath,
    threads        = 4,
    vaePath        = vaePath,
    t5xxlPath      = t5xxlPath,
    clipVisionPath = clipVisionPath,
)
check(ok) { "Failed to initialize video model" }
```

### Parameters

- `modelPath`: absolute or platform-resolved path to the model file
- `threads`: CPU threads to use. Pass `-1` to let the backend decide
- `vaePath`: path to the VAE file. Required for DiT-based video models (Wan, LTX); `null` otherwise
- `t5xxlPath`: path to the T5-XXL text encoder. Required for DiT-based video models; `null` otherwise
- `clipVisionPath`: path to the CLIP vision encoder. Required for DiT-based video models; `null` otherwise

You normally call this once and reuse the loaded model for multiple generations.

## `txt2img(...)`
Generates one image and returns the raw result as an **RGBA byte array**.

```kotlin
val rgba = StableDiffusionBridge.txt2img(
    prompt = "A cinematic photo of a llama astronaut",
    negativePrompt = "blurry, low quality, distorted",
    width = 512,
    height = 512,
    steps = 20,
    cfgScale = 7.5f,
    seed = 42L,
)
```

### Parameters

- `prompt`: what you want to generate
- `negativePrompt`: what you want the model to avoid
- `width` / `height`: output size in pixels
- `steps`: number of diffusion steps. Higher usually means slower but can improve quality
- `cfgScale`: how strongly the model should follow the prompt
- `seed`: controls reproducibility. Use a fixed value for repeatable output, or `-1L` for non-deterministic behavior

### Return value

The function returns an RGBA `ByteArray` of size `width * height * 4`.
If generation fails, the returned array may be empty.

## Converting the RGBA result

The bridge returns raw pixels so you can decide how to display or save them on each platform.

Typical workflow:
- Android: convert the bytes into a `Bitmap`
- iOS: convert into `UIImage` / CoreGraphics image data
- Desktop: convert into `BufferedImage`

Pseudo-code idea:

```kotlin
val rgba = StableDiffusionBridge.txt2img(...)
check(rgba.isNotEmpty())
// Convert width/height/RGBA bytes into a platform image type
```

## `img2img(...)`
Generates an image using an existing image as the starting point, guided by a text prompt.
This is useful for style transfer, inpainting workflows, or iterating on an existing image.

```kotlin
val sourceRgba: ByteArray = /* your RGBA source image bytes */
val rgba = StableDiffusionBridge.img2img(
    initImageRgba = sourceRgba,
    initImageW = 512,
    initImageH = 512,
    prompt = "A watercolor painting of the same scene",
    negativePrompt = "low quality, blurry",
    width = 512,
    height = 512,
    steps = 20,
    cfgScale = 7.0f,
    strength = 0.75f,
    seed = 42L,
)
```

### Parameters

- `initImageRgba`: the source image as a raw RGBA byte array (`width * height * 4` bytes)
- `initImageW` / `initImageH`: dimensions of the source image
- `prompt`: text describing the desired output
- `negativePrompt`: what the model should avoid
- `width` / `height`: output image size in pixels
- `steps`: number of diffusion steps
- `cfgScale`: prompt adherence strength
- `strength`: how much the source image influences the result. `0.0` preserves the original entirely; `1.0` ignores it. Values around `0.6–0.8` work well for most style transfers.
- `seed`: controls reproducibility. Use `-1L` for non-deterministic behavior.

### Return value

Returns an RGBA `ByteArray` of size `width * height * 4`.
Returns an empty array on failure.

## `release()`
Frees the native Stable Diffusion context.

```kotlin
StableDiffusionBridge.release()
```

Call this when you are done with image generation or when the feature is being torn down.

## `supportsVideoGeneration()`
Returns `true` if the currently loaded model supports video generation.
Must be called after `initModel`.

```kotlin
val ok = StableDiffusionBridge.initModel(modelPath, vaePath = vaePath, t5xxlPath = t5xxlPath, clipVisionPath = clipVisionPath)
val videoSupported = StableDiffusionBridge.supportsVideoGeneration()
```

Use this to conditionally show video generation UI or skip `txt2video` calls for image-only models.

## `txt2video(...)`
Generates video frames from a text prompt and returns them as a packed byte array.

```kotlin
val raw = StableDiffusionBridge.txt2video(
    prompt         = "A llama walking through a futuristic city",
    negativePrompt = "blurry, low quality",
    width          = 512,
    height         = 512,
    videoFrames    = 16,
    fps            = 8,
    cfgScale       = 7.0f,
    seed           = 42L,
)
```

### Parameters

- `prompt`: text describing the desired video content
- `negativePrompt`: what the model should avoid
- `width` / `height`: frame dimensions in pixels. Default `512×512`
- `videoFrames`: number of frames to generate. Default `16`. Higher values produce longer clips but take proportionally longer
- `fps`: playback frame rate. Default `8`. Does not affect generation time
- `cfgScale`: prompt adherence strength. Default `7.0`
- `seed`: controls reproducibility. Use `-1L` for non-deterministic behavior

### Return value

Returns a packed `ByteArray` with a 16-byte little-endian header followed by the raw RGBA frame data:

```
[frameCount : 4 bytes LE][width : 4 bytes LE][height : 4 bytes LE][fps : 4 bytes LE]
[frame 0 RGBA: width * height * 4 bytes]
[frame 1 RGBA: width * height * 4 bytes]
...
```

Returns an empty array on failure.

See the [Video generation guide](../../guides/video-generation/) for a complete parsing and gallery-save example.

## Practical recommendations

- Start with `512x512`, `steps = 20`, `cfgScale = 7.0f`.
- Use a negative prompt for better consistency when quality matters.
- Reuse one initialized model for multiple generations instead of loading it every time.
- For `img2img`, ensure your source RGBA byte array is correctly sized (`initImageW * initImageH * 4`).
- Expect significant CPU and memory use on mobile devices.
