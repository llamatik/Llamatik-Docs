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

SSE responses are never gzip/deflate compressed, so each delta reaches the client as soon as it is generated. If you run the server behind a reverse proxy (nginx, etc.), disable response buffering for streaming routes.

On error:

```
data: {"event":"error","message":"..."}
```

## Sections

- [Models Status](models/) — loaded models and capabilities for every bridge
- [Generation](generation/) — text generation, streaming, JSON output, KV sessions, MTP, model metadata
- [Embeddings](embeddings/) — embedding vectors
- [Whisper](whisper/) — speech-to-text
- [Stable Diffusion](stablediffusion/) — image and video generation
- [Multimodal](multimodal/) — vision-language streaming
