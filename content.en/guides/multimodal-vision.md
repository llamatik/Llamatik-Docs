---
title: "Multimodal vision (VLM)"
weight: 9
---

`MultimodalBridge` lets you analyze images on-device using a vision-language model (VLM).
The model can describe scenes, answer questions about images, extract text, and reason about visual content.

## Required models

You need two files from the same VLM release:

1. A GGUF language model (e.g. `llava-v1.6-mistral-7b.Q4_K_M.gguf`)
2. A multimodal projector (e.g. `llava-v1.6-mistral-7b-mmproj-model-f16.gguf`)

The projector must match the language model — mixing files from different releases will produce incorrect or empty output.

## Initialize the model

```kotlin
val modelPath = LlamaBridge.getModelPath("llava-v1.6-mistral-7b.Q4_K_M.gguf")
val mmprojPath = LlamaBridge.getModelPath("llava-v1.6-mistral-7b-mmproj-model-f16.gguf")

val ok = MultimodalBridge.initModel(modelPath, mmprojPath)
check(ok) { "Failed to initialize VLM" }
```

## Check what the model supports

Not every multimodal model accepts images — some are audio-only or speech models. After loading, ask the bridge what the model can do:

```kotlin
val caps = MultimodalBridge.getCapabilities()

if (!caps.canAnalyzeImages) {
    showMessage("The loaded model can't analyze images.")
    return
}
```

Use this to show or hide the image-attach button, and to show a clear message instead of a generic error. See [`MultimodalBridge`]({{< relref "../api/multimodalbridge" >}}) for every field.

## Analyze an image

`analyzeImageBytesStream` accepts raw image bytes (JPEG, PNG, or BMP) and a text prompt.
It streams the response token by token using a `GenStream` callback.

The call blocks until the stream completes — always call it from a background thread.

```kotlin
val imageBytes: ByteArray = /* JPEG or PNG bytes from camera, gallery, or file */

MultimodalBridge.analyzeImageBytesStream(
    imageBytes = imageBytes,
    prompt = "Describe what you see in this image.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(message: String) = println("Error: $message")
    }
)
```

## Practical prompt examples

Ask targeted questions to get focused answers:

```kotlin
// Identify objects
prompt = "List all the objects you can see in this image."

// Read text
prompt = "Transcribe any text visible in this image."

// Describe for accessibility
prompt = "Write a concise alt-text description of this image."

// Answer a specific question
prompt = "Is there a person in this image? Describe what they are doing."
```

## Cancelling analysis

If the user leaves the screen or taps Stop, cancel the running analysis:

```kotlin
MultimodalBridge.cancelAnalysis()
```

Safe to call from any thread. The current stream will stop and the callback will be finalized.

## Cleanup

```kotlin
MultimodalBridge.release()
```

Release resources when leaving the vision feature or shutting the app down.
Do not call while `analyzeImageBytesStream` is still running.

## Practical notes

- Load the model once and reuse it for multiple analyses.
- VLMs are significantly larger than pure language models — check available memory before loading.
- Use a quantized model (Q4_K_M or similar) for a practical mobile footprint.
- The mmproj file is typically F16 and does not compress further.
- WASM support requires a separate Web Worker setup and is not available out of the box.
