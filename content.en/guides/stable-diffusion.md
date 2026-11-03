---
title: "Stable Diffusion image generation"
weight: 7
---

`StableDiffusionBridge` lets you generate images locally from text prompts.
The bridge returns raw RGBA pixel data so your app can convert it into the most appropriate image type for the current platform.

## Initialize the model

```kotlin
// Standard model (SD, SDXL, …)
val modelPath = StableDiffusionBridge.getModelPath("dreamshaper.safetensors")
val ok = StableDiffusionBridge.initModel(modelPath, threads = 4)
check(ok)

// DiT video model (Wan, LTX) — pass companion file paths
val ok = StableDiffusionBridge.initModel(
    modelPath      = StableDiffusionBridge.getModelPath("wan-video.gguf"),
    threads        = 4,
    vaePath        = StableDiffusionBridge.getModelPath("wan-vae.safetensors"),
    t5xxlPath      = StableDiffusionBridge.getModelPath("t5xxl_fp16.safetensors"),
    clipVisionPath = StableDiffusionBridge.getModelPath("clip_vision.safetensors"),
)
check(ok)
```

## Generate an image

```kotlin
val rgba = StableDiffusionBridge.txt2img(
    prompt = "A cinematic illustration of a llama explorer",
    negativePrompt = "blurry, low quality, extra limbs",
    width = 512,
    height = 512,
    steps = 20,
    cfgScale = 7.0f,
    seed = 1234L,
)
```

## Understanding the result

The output is an RGBA `ByteArray`.
To display it, convert the bytes into your platform image type using the same width and height you requested.

## Image-to-image generation

`img2img` generates a new image starting from an existing one, guided by a text prompt.
The source image must be provided as a raw RGBA byte array.

```kotlin
val sourceRgba: ByteArray = /* existing image bytes as RGBA */
val result = StableDiffusionBridge.img2img(
    initImageRgba = sourceRgba,
    initImageW = 512,
    initImageH = 512,
    prompt = "The same scene as a watercolor painting",
    negativePrompt = "low quality, blurry",
    width = 512,
    height = 512,
    steps = 20,
    cfgScale = 7.0f,
    strength = 0.75f,
    seed = 1234L,
)
```

The `strength` parameter controls how much the source image is preserved:

- `0.0` leaves the image almost unchanged
- `1.0` ignores the source entirely and acts like `txt2img`
- values around `0.6–0.8` are a good starting point for style transfers

Make sure the `initImageRgba` array contains exactly `initImageW * initImageH * 4` bytes.

## Parameter guidance

- `width` and `height`: start with `512x512` unless you have benchmarked larger values
- `steps`: around 20 is a good mobile-friendly default
- `cfgScale`: around 7.0 to 8.0 is a common starting range
- `seed`: use a fixed value for reproducible results
- `strength` (`img2img` only): start around 0.75 and adjust to taste

## Performance advice

Image generation is significantly heavier than text generation.
On mobile devices:
- prefer moderate output sizes
- avoid excessive step counts for the first iteration
- reuse one initialized model for many generations

## Cleanup

```kotlin
StableDiffusionBridge.release()
```

## Video generation

`StableDiffusionBridge` also supports text-to-video generation for DiT-based models such as Wan and LTX.
See the [Video generation guide](../video-generation/) for full details on `txt2video`, frame parsing, and saving clips to the device gallery.
