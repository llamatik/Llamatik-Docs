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
    fun setProgressListener(listener: SdProgressListener?)
    fun loadControlNet(path: String): Boolean
    fun unloadControlNet(): Boolean
    fun hasControlNet(): Boolean
    fun generateWithControl(
        controlImageRgba: ByteArray,
        controlImageW: Int,
        controlImageH: Int,
        controlType: SdControlType,
        prompt: String,
        negativePrompt: String? = null,
        width: Int = 512,
        height: Int = 512,
        steps: Int = 20,
        cfgScale: Float = 7.0f,
        controlStrength: Float = 0.9f,
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

## `setProgressListener(listener)`
Registers a listener that receives progress while `txt2img`, `img2img`, `generateWithControl` or `txt2video` is running.
Pass `null` to clear it.

```kotlin
StableDiffusionBridge.setProgressListener { p ->
    val percent = p.fraction?.let { (it * 100).toInt() }   // null when the total is unknown
    println("${p.stage}: ${p.completed}/${p.total} ${p.unit ?: ""} ($percent%)")
}

try {
    val rgba = StableDiffusionBridge.txt2img(prompt = "a lighthouse at dusk")
} finally {
    StableDiffusionBridge.setProgressListener(null)
}
```

- The listener is called **synchronously on the thread running the generation**. Hand off to your own dispatcher before touching UI.
- Register or clear it only while no generation is running.
- The listener survives `initModel` and `release`.
- On WASM this is a no-op.

### `SdProgress`

```kotlin
data class SdProgress(
    val stage: String,       // library stage name, e.g. text encoding, sampling, decoding
    val detail: String?,
    val unit: String?,       // unit of completed/total, e.g. steps
    val samplePass: Int,     // counted from 1 since the listener was registered
    val completed: Long,
    val total: Long,         // 0 = unknown
) {
    val fraction: Float?     // 0f..1f, null when total is unknown
}
```

`stage`, `detail` and `unit` come straight from stable-diffusion.cpp and are not localized. Prefer showing `fraction` (a percentage) in user-facing UI.

## ControlNet

A ControlNet model steers generation with a control image (a pose, a depth map, canny edges, ...).

### `loadControlNet(path)` / `unloadControlNet()` / `hasControlNet()`
`loadControlNet` loads a ControlNet model into the **current** context (hot-swap — it replaces any loaded one).
Call `initModel` first, and never call these during a generation.
`initModel` may create a new context (always on iOS; on Android and desktop whenever the model files change), which drops the loaded ControlNet.
After every `initModel`, check `hasControlNet()` and load the ControlNet again if needed.

```kotlin
check(StableDiffusionBridge.initModel(modelPath))
check(StableDiffusionBridge.loadControlNet(controlNetPath)) { "Failed to load ControlNet" }
```

### `generateWithControl(...)`
Generates an image guided by a control image.
It requires a loaded ControlNet and returns an empty array otherwise.

```kotlin
val rgba = StableDiffusionBridge.generateWithControl(
    controlImageRgba = edgesRgba,         // RGBA, controlImageW * controlImageH * 4 bytes
    controlImageW = 512,
    controlImageH = 512,
    controlType = SdControlType.CANNY,
    prompt = "a modern glass house, golden hour",
    controlStrength = 0.9f,
)
```

#### Parameters

- `controlImageRgba`, `controlImageW`, `controlImageH`: the control image, RGBA. It must already be in the form the ControlNet expects (for example an edge map for `CANNY`); Llamatik does not preprocess it.
- `controlType`: an `SdControlType` that must match what the ControlNet model was trained for.
- `controlStrength`: how strongly the control image steers the result. `0.9` is a sensible default.
- the remaining parameters behave like [`txt2img`](#txt2img).

#### `SdControlType`

| Value | Use for |
|---|---|
| `NONE` | No control |
| `OPENPOSE` | Human pose |
| `DEPTH` | Depth maps |
| `SCRIBBLE` | Thick lines: scribble, HED, soft edge, TED |
| `CANNY` | Thin lines: canny, lineart, anime lineart, MLSD |
| `NORMAL` | Normal maps |
| `SEGMENT` | Segmentation maps |
| `TILE` | Tiling / detail refinement |
| `REPAINT` | Inpaint, outpaint |

The return value is RGBA, the same as `txt2img`. See [Converting the RGBA result](#converting-the-rgba-result).

## Practical recommendations

- Start with `512x512`, `steps = 20`, `cfgScale = 7.0f`.
- Use a negative prompt for better consistency when quality matters.
- Reuse one initialized model for multiple generations instead of loading it every time.
- For `img2img`, ensure your source RGBA byte array is correctly sized (`initImageW * initImageH * 4`).
- Expect significant CPU and memory use on mobile devices.
