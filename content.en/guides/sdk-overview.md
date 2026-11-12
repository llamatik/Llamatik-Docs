---
title: "SDK overview (v2.0)"
weight: 13
---

Llamatik 2.0 introduces `com.llamatik:sdk` — a module that extracts all reusable business logic
from the application layer into a clean, UI-agnostic library with zero Compose dependencies.

## Module structure

```
:core   → LlamaBridge, WhisperBridge, StableDiffusionBridge, MultimodalBridge (JNI bridges)
  ↑
:sdk    → chat, models, download, RAG, agents, assistant, http
  ↑
:shared → Compose UI, ViewModels, navigation, DI
```

Add it to your project:

```kotlin
implementation("com.llamatik:sdk:<version>")
```

## Chat & Prompts

`com.llamatik.sdk.chat` contains everything needed to build a structured chat experience:

| Type | Description |
|---|---|
| `ChatMessage` | A single message with role and content |
| `PromptTemplate` | Template variant: `Plain`, `Gemma3`, `Llama3Instruct`, `QwenChat` |
| `PromptRenderer` | Renders system + context + messages into a model-ready prompt string |
| `ChatRunner` | Orchestrates a streaming chat turn using the active `LlamaSession` |
| `ChatHistoryRepository` | Persists and retrieves conversation history |

```kotlin
import com.llamatik.sdk.chat.*

val labels = PromptRenderer.Labels(
    system = "System",
    user = "User",
    assistant = "Assistant",
    relevantContext = "Relevant context",
)

val prompt = PromptRenderer.render(
    systemPrompt = "You are a helpful assistant.",
    contexts = emptyList(),
    messages = listOf(ChatMessage(role = "user", content = "Hello!")),
    template = Llama3Instruct,
    labels = labels,
)
```

## Model management

`com.llamatik.sdk.model` provides model discovery, storage, and lifecycle management:

| Type | Description |
|---|---|
| `LlamaModel` | Metadata for a GGUF model (name, path, source, size) |
| `ModelSource` | `Local` or `Remote` (URL + progress) |
| `GenerateSettings` | Serializable generation parameter set |
| `ModelsRepository` | CRUD for the local model catalog |
| `GetModelsUseCase` | Retrieves the current model list |
| `ImportModelUseCase` | Imports a model from a file picker or URL |

```kotlin
import com.llamatik.sdk.model.*

val repo = ModelsRepository(fileAccess = myFileAccess, httpClient = LlamatikHttpClient())
val models: List<LlamaModel> = GetModelsUseCase(repo).execute()
```

## Download orchestration

`com.llamatik.sdk.download` manages model downloads with progress events:

```kotlin
import com.llamatik.sdk.download.*

val orchestrator: ModelDownloadOrchestrator = DefaultModelDownloadOrchestrator(httpClient, fileAccess)

orchestrator.download(url, destinationPath).collect { event ->
    when (event) {
        is DownloadEvent.Progress  -> updateProgress(event.bytesReceived, event.totalBytes)
        is DownloadEvent.Completed -> onDownloadComplete(event.path)
        is DownloadEvent.Failed    -> showError(event.message)
    }
}
```

## RAG / Vector Store

`com.llamatik.sdk.rag` provides everything needed for retrieval-augmented generation:

| Function / Type | Description |
|---|---|
| `VectorStoreItem` | A stored document chunk with its embedding vector |
| `PersistedRagStore` | Serializable vector store backed by `Settings` |
| `chunkText(text, chunkSize)` | Splits a document into overlapping chunks |
| `retrieveContext(query, store, topK)` | Returns the most relevant chunks for a query |
| `cosineD(a, b)` | Cosine distance between two embedding vectors |
| `mmr(candidates, selected, lambda)` | Maximal Marginal Relevance reranking |

```kotlin
import com.llamatik.sdk.rag.*

// Index a document
val chunks = chunkText(documentText, chunkSize = 512)
val items = chunks.map { chunk ->
    VectorStoreItem(
        text = chunk,
        embedding = LlamaBridge.embed(chunk),
    )
}

// Retrieve at query time
val queryEmbedding = LlamaBridge.embed(userQuery)
val topChunks = retrieveContext(queryEmbedding, store = ragStore, topK = 5)
```

## Assistant facade

`com.llamatik.sdk.assistant.Assistant` is a single injectable facade that owns all inference orchestration — chat, RAG, image generation, VLM, STT, model download, and session persistence.

Inject it via Koin or your own DI:

```kotlin
// Koin registration (done in CommonModule)
factory { (scope: String, systemPrompt: String?) -> Assistant(scope, systemPrompt, ...) }

// Usage
val assistant = get<Assistant> { parametersOf("chat", null) }
assistant.streamChat(message).collect { chunk -> render(chunk) }
```

### Vision model capabilities

After a VLM is loaded, `AssistantState.vlmCapabilities` holds what the model can do. It is `MultimodalCapabilities.NONE` when no VLM is loaded, and it resets when the model is unloaded. `assistant.multimodalCapabilities()` returns the same value.

```kotlin
val caps = assistant.multimodalCapabilities()
showAttachImageButton = caps.canAnalyzeImages
```

`assistant.analyzeImage(...)` checks `canAnalyzeImages` before running. If the loaded model can't analyze images, it posts `visionUnsupportedMessage` (a new optional parameter) as the bot reply instead of calling the model.

### Image generation progress

While an image is being generated, `AssistantState.generationProgress` holds the live `SdProgress` (or `null` when idle or unknown). Use `progress.fraction` for a progress bar.

### ControlNet and video upscaling

```kotlin
// ControlNet — a Stable Diffusion model must be loaded first
assistant.loadControlNet(controlNetPath)               // suspend; updates state.isControlNetLoaded
assistant.generateImageWithControl(
    prompt = "a modern glass house",
    controlImageBytes = edgesPng,
    controlType = SdControlType.CANNY,
    decodeImageBytesToRgba = { bytes -> decodeToRgba(bytes) },
)

// Video upscaling — independent from the generation model
assistant.loadVideoUpscaler(upscalerModelPath, upscalerVaePath, upscaleFactor = 2)
val upscaled = assistant.upscaleVideo(packedFrames)    // suspend; empty array on failure
assistant.unloadVideoUpscaler()
```

`state.isControlNetLoaded` and `state.isVideoUpscalerLoaded` reflect what is loaded.
