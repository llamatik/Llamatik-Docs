---
title: "Llamatik Docs"
weight: 1
bookFlatSection: true
---
# Llamatik

**Kotlin-first `llama.cpp` integration** for on-device and remote LLM inference.

- Kotlin Multiplatform: Android, iOS, desktop (JVM) and more
- Offline inference via bundled native bindings
- Embeddings + text generation + streaming
- **Schema‑constrained JSON generation** (optional JSON Schema → valid JSON output)
- **v2.0** — `:sdk` module, Agent Framework DSL, Agent Platform runtime

## What's new in v2.0

| Feature | Module |
|---|---|
| Clean SDK module with zero UI dependencies | `com.llamatik:sdk` |
| `Assistant` facade — single injectable class for all inference | `com.llamatik:sdk` |
| `Agent { }` Kotlin DSL — streaming, memory, tools, workflows | `com.llamatik:sdk` |
| Agent Platform — runtime, permissions, companion profiles, audit | `com.llamatik:sdk-agent` |

→ **[Migration Guide](migration/)** for upgrading from 1.x

## Quick links

- **Getting started:** head to [Installation](getting-started/installation/) and [Quickstart](getting-started/quickstart/)
- **Guides:** [SDK overview](guides/sdk-overview/), [Agent Framework](guides/agent-framework/), [Agent Platform](guides/agent-platform/), embeddings, streaming, JSON mode
- **API reference:** `LlamaBridge`, `GenStream`, SDK types
- **Llamatik Server:** [self-hosted HTTP backend](llamatik-server/) — run all inference bridges over HTTP
- **Troubleshooting:** common build/linking pitfalls (especially iOS)
- **Legacy (v1.x):** [v1.x documentation](v1-legacy/)

## Llamatik Server

[**Llamatik Server**](llamatik-server/) is a self-hosted Ktor backend that exposes the complete `com.llamatik:core` API over HTTP. One server, all inference — generation, embeddings, Whisper, Stable Diffusion image/video, and multimodal VLM.

- [Getting Started](llamatik-server/getting-started/) — run the server, first `curl`
- [Configuration](llamatik-server/configuration/) — env vars, JWT auth, path security
- [API Reference](llamatik-server/api/) — all endpoints by bridge
- [Deployment](llamatik-server/deployment/) — Docker, systemd, HTTPS

## Llamatik Code — IntelliJ Plugin

[**Llamatik Code**](llamatik-code/) is a local-first AI coding assistant for IntelliJ-based IDEs built on top of this library.

- [Getting Started](llamatik-code/getting-started/) — install, configure, first chat
- [Agent & Tools](llamatik-code/agent-and-tools/) — agent loop, built-in and git tools
- [MCP Servers](llamatik-code/mcp-servers/) — connect external tool servers (1.4.0+)
- [External Knowledge](llamatik-code/external-knowledge/) — web search, docs research, citations (1.5.0+)
- [Code Health Monitor](llamatik-code/code-health-monitor/) — background code quality analysis (1.5.0+)
- [Settings Reference](llamatik-code/settings-reference/) — all configuration options
- [Changelog](llamatik-code/changelog/) — version history from 1.2.0 to 1.5.0

> Repo: https://github.com/ferranpons/Llamatik/
