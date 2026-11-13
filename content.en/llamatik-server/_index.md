---
title: "Llamatik Server"
weight: 40
bookCollapseSection: true
---

# Llamatik Server

**Llamatik Server** is a self-hosted HTTP backend that exposes the complete Llamatik library API over HTTP. Deploy it on any JVM machine and point your Llamatik apps — or any HTTP client — at it.

All endpoints mirror the bridge surface of `com.llamatik:core` 2.2.0:

| Bridge | Route prefix | Capabilities |
|---|---|---|
| `LlamaBridge` | `/v1/generation` | Text generation, streaming SSE, JSON output, KV sessions, MTP speculative decoding |
| `LlamaBridge` | `/v1/embeddings` | Embedding vectors for RAG and search |
| `WhisperBridge` | `/v1/whisper` | Speech-to-text, per-segment timestamps, speaker turns |
| `StableDiffusionBridge` | `/v1/stablediffusion` | Text-to-image, image-to-image, text-to-video (Wan 2.1/2.2) |
| `MultimodalBridge` | `/v1/multimodal` | Vision-language streaming analysis |

## Typical use cases

- **Shared inference** — one powerful machine running models; many apps calling the same server
- **Hybrid mode** — apps run on-device by default and fall back to this server when the device is too slow or the model too large
- **Heavy workloads** — GPU-backed multimodal or video generation on a dedicated machine, consumed by any client over the network

## Using it from the Llamatik app

The Llamatik app can discover and connect to a server from **Settings → Connect to Server**, stream responses from it, and show its status in the chat toolbar. See [Connecting to a server](../guides/connecting-to-a-server/).

> **Connections are currently unencrypted (plain HTTP).** Do not send sensitive data over untrusted networks. Encrypted connections are planned; see [Deployment](deployment/#https-with-a-reverse-proxy) for TLS via a reverse proxy.

## Quick links

- **[Getting started](getting-started/)** — run the server, load a model, make your first request
- **[Configuration](configuration/)** — env vars, authentication, path security
- **[API Reference](api/)** — complete endpoint documentation
- **[Deployment](deployment/)** — Docker, systemd, HTTPS

> Source: https://github.com/ferranpons/Llamatik-Server/
