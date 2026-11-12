---
title: "VideoUpscalerBridge"
weight: 4
---

`VideoUpscalerBridge` upscales video on-device with a video super-resolution model (RelayVSR / SeedVR2 style), powered by stable-diffusion.cpp.
Use it on clips produced by [`StableDiffusionBridge.txt2video`]({{< relref "stablediffusionbridge" >}}) or on your own frames.

It owns its own native context, independent from `StableDiffusionBridge`: you can load it while a generation model is loaded, or after releasing it.

## Platform support

- **Android**: supported
- **iOS**: supported
- **JVM / Desktop**: supported
- **WASM**: not available (every call fails or returns an empty array)

## Public API

```kotlin
expect object VideoUpscalerBridge {
    fun initModel(
        diffusionModelPath: String,
        vaePath: String,
        upscaleFactor: Int = 2,
        threads: Int = -1,
    ): Boolean
    fun isLoaded(): Boolean
    fun upscale(
        packedFrames: ByteArray,
        seed: Long = -1L,
        windowSize: Int = 0,
        temporalChunkSize: Int = 0,
    ): ByteArray
    fun release()
}
```

## `initModel(...)`
Loads the upscaler model and its VAE.

- `diffusionModelPath`, `vaePath`: the upscaler's diffusion model and its matching VAE.
- `upscaleFactor`: `2` or `4`. RelayVSR requires it to match both checkpoints.
- `threads`: CPU threads, `-1` for the default.

Calling `initModel` again replaces the previously loaded upscaler.

## `upscale(...)`
Upscales a packed frame sequence and returns it in the same layout. It blocks until finished — call it from a background thread.

The layout is the one `txt2video` returns:

- a 16-byte little-endian header: `frameCount`, `width`, `height`, `fps` (each an `Int32`)
- followed by `frameCount * width * height * 4` bytes of RGBA frames

All frames must have the same size. The output header carries the **new** `width` and `height`.

```kotlin
check(VideoUpscalerBridge.initModel(modelPath, vaePath, upscaleFactor = 2))

val packed = StableDiffusionBridge.txt2video(prompt = "a paper boat on a river")
val upscaled = VideoUpscalerBridge.upscale(packed)
check(upscaled.isNotEmpty()) { "Upscaling failed" }

// Parse `upscaled` exactly like a txt2video result.
```

### Parameters

- `seed`: negative keeps the library default.
- `windowSize`: RelayVSR reference window, including both endpoints. `0` uses the library default (`16`).
- `temporalChunkSize`: SeedVR2 independent temporal chunk, in input frames (`1` or `4n+1`). `0` uses the library default (`5`).

Returns an empty array on failure.

## `isLoaded()` / `release()`
`isLoaded()` is `true` after a successful `initModel`. `release()` frees the upscaler.

## Practical notes

- Upscaling is heavy: memory grows with frame count and output size. Upscale short clips, or in chunks, on mobile.
- The model files are separate from your generation model; download them like any other model.
- Progress reporting is not available for the upscaler.
