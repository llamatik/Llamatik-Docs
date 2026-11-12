---
title: "MultimodalBridge"
weight: 5
---

`MultimodalBridge` provides on-device vision-language model (VLM) inference.
It combines a language model with a multimodal projector to analyze images and answer questions about them.

## Platform support

- **Android**: supported
- **iOS**: supported
- **JVM / Desktop**: supported
- **WASM**: model paths are stored, but analysis is dispatched to a Web Worker — the worker must be wired up separately

## Public API

```kotlin
expect object MultimodalBridge {
    fun initModel(modelPath: String, mmprojPath: String): Boolean
    fun analyzeImageBytesStream(
        imageBytes: ByteArray,
        prompt: String,
        callback: GenStream,
    )
    fun getCapabilities(): MultimodalCapabilities
    fun cancelAnalysis()
    fun release()
}
```

## `initModel(modelPath, mmprojPath)`
Loads both the language model and the multimodal projector (mmproj).
Returns `true` on success.

```kotlin
val modelPath = LlamaBridge.getModelPath("llava-v1.6-mistral-7b.Q4_K_M.gguf")
val mmprojPath = LlamaBridge.getModelPath("llava-v1.6-mistral-7b-mmproj-model-f16.gguf")

val ok = MultimodalBridge.initModel(modelPath, mmprojPath)
check(ok) { "Failed to initialize VLM model" }
```

Both the main model and the mmproj file must come from the same model family and release.

## `analyzeImageBytesStream(imageBytes, prompt, callback)`
Analyzes an image and streams the model's response token by token.
The image can be provided as JPEG, PNG, or BMP bytes — the bridge handles decoding.

This method blocks until the stream completes.
Always call it from a background thread.

```kotlin
val imageBytes: ByteArray = /* JPEG or PNG bytes */

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

### Parameters

- `imageBytes`: raw image file bytes (JPEG, PNG, or BMP)
- `prompt`: the question or instruction about the image
- `callback`: a [GenStream]({{< relref "genstream" >}}) instance that receives tokens, completion, and error events

### Notes

- Call from a background thread — this call blocks for the duration of the stream.
- Use `cancelAnalysis()` to interrupt a running analysis.

## `getCapabilities()`
Returns what the currently loaded model and projector pair can do, as reported by llama.cpp (`mtmd_helper_get_capabilities`).
Returns `MultimodalCapabilities.NONE` (every flag `false`) when no model is loaded.

```kotlin
val caps = MultimodalBridge.getCapabilities()

if (caps.canAnalyzeImages) {
    // safe to call analyzeImageBytesStream
} else {
    showMessage("The loaded model can't analyze images.")
}
```

Call it after `initModel` succeeds. The result is cheap to compute, but it does not change until you load a different model.

### `MultimodalCapabilities`

```kotlin
data class MultimodalCapabilities(
    val chat: Boolean,
    val vision: Boolean,
    val audio: Boolean,
    val video: Boolean,
    val audioStream: Boolean,
    val textGen: Boolean,
    val audioGen: Boolean,
    val audioGenSpeakerRef: Boolean,
) {
    val canAnalyzeImages: Boolean  // chat && vision && textGen
    companion object { val NONE: MultimodalCapabilities }
}
```

| Field | Meaning |
|---|---|
| `chat` | The model can be used for chat. `false` for TTS-only and realtime transcription models. |
| `vision` | Accepts image input. |
| `audio` | Accepts audio input. |
| `video` | Accepts video input. Also depends on how llama.cpp was built. |
| `audioStream` | Supports realtime audio transcription. |
| `textGen` | Produces text output. `false` for audio-only models. |
| `audioGen` | Produces audio output (speech). |
| `audioGenSpeakerRef` | Accepts a speaker reference audio for voice cloning. |
| `canAnalyzeImages` | Convenience: `chat && vision && textGen`. |

{{< hint info >}}
Llamatik currently exposes image analysis only. The `audio`, `video`, `audioStream` and `audioGen` flags tell you what the model supports; they do not mean Llamatik offers an API for them yet.
{{< /hint >}}

### Platform notes

- **Android, iOS, JVM / Desktop**: the values come from the native model.
- **WASM**: the model is loaded lazily inside the Web Worker, so real values are not available. After `initModel` the bridge reports `chat`, `vision` and `textGen` as `true`; before it, `NONE`.

## `cancelAnalysis()`
Cancels a running `analyzeImageBytesStream` call.

```kotlin
MultimodalBridge.cancelAnalysis()
```

Safe to call from any thread.
The in-progress stream will stop and `onComplete` or `onError` will be called to finalize the callback.

## `release()`
Frees the model, the multimodal projector context, and the language model context.

```kotlin
MultimodalBridge.release()
```

Call this when leaving the vision feature or when the app is shutting down.
Do not call while `analyzeImageBytesStream` is still running.
