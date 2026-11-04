---
title: "API Reference"
weight: 30
bookCollapseSection: true
---

# API Reference

All endpoints are prefixed with `/v1`. Requests and responses use `Content-Type: application/json` unless noted.

Streaming endpoints respond with `Content-Type: text/event-stream`. Each SSE event is a JSON line:

```
data: {"event":"delta","text":"..."}
data: {"event":"done"}
```

On error:

```
data: {"event":"error","message":"..."}
```

## Sections

- [Generation](generation/) — text generation, streaming, JSON output, KV sessions, MTP, model metadata
- [Embeddings](embeddings/) — embedding vectors
- [Whisper](whisper/) — speech-to-text
- [Stable Diffusion](stablediffusion/) — image and video generation
- [Multimodal](multimodal/) — vision-language streaming
