---
title: "Video generation"
weight: 8
---

`StableDiffusionBridge` supports text-to-video generation with DiT-based models such as Wan and LTX.
`supportsVideoGeneration()` tells you at runtime whether the currently loaded model supports it, so you can conditionally expose the feature without hardcoding model names.

## Check for video support

Always call `supportsVideoGeneration()` after `initModel` before offering video generation to users.

```kotlin
val ok = StableDiffusionBridge.initModel(
    modelPath = modelPath,
    vaePath = vaePath,
    t5xxlPath = t5xxlPath,
    clipVisionPath = clipVisionPath,
)
check(ok)

val videoSupported = StableDiffusionBridge.supportsVideoGeneration()
```

## Initialize a video model

DiT-based video models (Wan, LTX) require companion files in addition to the main model file.
Pass them as optional parameters to `initModel`.

```kotlin
val modelPath       = StableDiffusionBridge.getModelPath("wan-video.gguf")
val vaePath         = StableDiffusionBridge.getModelPath("wan-vae.safetensors")
val t5xxlPath       = StableDiffusionBridge.getModelPath("t5xxl_fp16.safetensors")
val clipVisionPath  = StableDiffusionBridge.getModelPath("clip_vision.safetensors")

val ok = StableDiffusionBridge.initModel(
    modelPath      = modelPath,
    threads        = 4,
    vaePath        = vaePath,
    t5xxlPath      = t5xxlPath,
    clipVisionPath = clipVisionPath,
)
check(ok) { "Failed to initialize video model" }
```

## Generate video frames

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

## Parse the result

`txt2video` returns a packed `ByteArray`.
The first 16 bytes are a little-endian header followed by the raw RGBA frame data.

```
[frameCount : 4 bytes LE][width : 4 bytes LE][height : 4 bytes LE][fps : 4 bytes LE]
[frame 0 RGBA bytes: width * height * 4]
[frame 1 RGBA bytes: width * height * 4]
...
```

Kotlin unpacking example:

```kotlin
import java.nio.ByteBuffer
import java.nio.ByteOrder

fun parseVideoResult(raw: ByteArray): List<ByteArray> {
    if (raw.isEmpty()) return emptyList()

    val header = ByteBuffer.wrap(raw, 0, 16).order(ByteOrder.LITTLE_ENDIAN)
    val frameCount = header.int
    val width      = header.int
    val height     = header.int
    // fps is at header.int — read if needed

    val frameSize = width * height * 4
    val frames = mutableListOf<ByteArray>()
    for (i in 0 until frameCount) {
        val offset = 16 + i * frameSize
        frames.add(raw.copyOfRange(offset, offset + frameSize))
    }
    return frames
}

val frames = parseVideoResult(raw)
```

If `raw` is empty, generation failed.

## Save to device gallery

`VideoSaver` (from the `shared` module) encodes and saves the frames as an H.264 video to the device gallery.

```kotlin
import com.llamatik.app.platform.VideoSaver

val saved = VideoSaver.saveVideo(
    frames = frames,
    width  = 512,
    height = 512,
    fps    = 8,
)
if (saved) println("Video saved to gallery")
```

`saveVideo` is a `suspend` function — call it from a coroutine.

## Upscale a generated clip

`VideoUpscalerBridge` increases the resolution of a clip with a video super-resolution model.
It works on the same packed result `txt2video` returns, and returns the same layout with the new size in its header.

```kotlin
VideoUpscalerBridge.initModel(upscalerModelPath, upscalerVaePath, upscaleFactor = 2)

val upscaled = VideoUpscalerBridge.upscale(packedBytes)   // call from a background thread
```

Parse `upscaled` exactly as shown above. See [`VideoUpscalerBridge`]({{< relref "../api/videoupscalerbridge" >}}) for the options and platform support.

## Parameter guidance

| Parameter | Default | Notes |
|---|---|---|
| `width` / `height` | `512` | Start with `512×512`; larger sizes increase memory and generation time significantly |
| `videoFrames` | `16` | Number of frames. Higher = longer clip, but proportionally slower |
| `fps` | `8` | Playback frame rate. Does not affect generation speed |
| `cfgScale` | `7.0` | Prompt adherence. `6.0–8.0` is a typical starting range |
| `seed` | `-1L` | Use a fixed value for reproducible clips, `-1L` for random |

## Platform notes

| Platform | Video generation | Gallery save |
|---|---|---|
| Android | Supported | MediaCodec H.264 + MediaStore |
| iOS | Supported | AVAssetWriter H.264 + PHPhotoLibrary |
| JVM / Desktop | Stub — returns `false` | Not available |
| WASM | Stub — returns `false` | Not available |

**iOS**: add `NSPhotoLibraryAddUsageDescription` to your `Info.plist` before calling `VideoSaver.saveVideo`, otherwise the save call will fail silently.
