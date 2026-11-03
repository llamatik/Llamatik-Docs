---
title: "API reference"
weight: 30
---

## Core bridges (`com.llamatik:core`)

Low-level JNI bridges — unchanged from v1.x.

| Type | Description |
|---|---|
| `LlamaBridge` | Text generation, embeddings, streaming, JSON output, KV sessions |
| `LlamaSession` | Isolated concurrent inference session with its own KV cache |
| `MultimodalBridge` | On-device vision-language model (VLM) image analysis |
| `StableDiffusionBridge` | On-device text-to-image, image-to-image, and text-to-video generation |
| `WhisperBridge` | On-device speech-to-text from WAV audio |
| `GenStream` | Callback interface for streaming token output |

## SDK types (`com.llamatik:sdk`)

| Package | Key types |
|---|---|
| `com.llamatik.sdk.chat` | `ChatMessage`, `PromptTemplate`, `PromptRenderer`, `ChatRunner`, `ChatHistoryRepository` |
| `com.llamatik.sdk.model` | `LlamaModel`, `GenerateSettings`, `ModelsRepository`, `GetModelsUseCase`, `ImportModelUseCase` |
| `com.llamatik.sdk.download` | `ModelDownloadOrchestrator`, `DownloadEvent` |
| `com.llamatik.sdk.rag` | `VectorStoreItem`, `PersistedRagStore`, `chunkText()`, `retrieveContext()`, `cosineD()`, `mmr()` |
| `com.llamatik.sdk.assistant` | `Assistant` |
| `com.llamatik.sdk.framework` | `Agent`, `Memory`, `Workflow`, `Tool`, `AgentEvent`, `AgentStreamChunk` |

## Agent Platform (`com.llamatik:sdk-agent`)

| Package | Key types |
|---|---|
| `com.llamatik.sdk.agent.runtime` | `AgentRuntime`, `AgentRuntime.Builder`, `AgentResponse`, `AgentContext` |
| `com.llamatik.sdk.agent.profile` | `CompanionProfile`, `CompanionConfig` |
| `com.llamatik.sdk.agent.permission` | `PermissionSet`, `PermissionGate`, `ToolId` |
| `com.llamatik.sdk.agent.audit` | `AuditLog`, `AuditEntry` |
| `com.llamatik.sdk.agent.memory` | `AgentMemory` |
| `com.llamatik.sdk.agent.tools` | `CalculatorTool`, `DateTimeTool`, `UuidTool`, `RandomTool`, `WebSearchTool`, `WebFetchTool`, `FileSystemTool`, `TranslationTool`, `SummarizerTool` |

If you are new to the library, start with **Getting started** first, then come back here for the exact function reference and usage patterns.
