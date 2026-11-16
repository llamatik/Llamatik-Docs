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

## Resumable streams

Streaming responses survive dropped connections. The generation keeps running on the server and every event is recorded, so a client that lost its connection can reattach and receive what it missed.

| Header | Direction | Meaning |
|---|---|---|
| `X-Stream-Id` | request | Client-chosen id (max 128 chars) for this stream. Send the same id again to reattach. |
| `Last-Event-ID` | request | Number of events the client already received. Only events after it are replayed. |
| `X-Stream-Resume: 1` | response | The server supports resuming this stream. |

Events carry a 1-based `id:` line:

```
id: 1
data: {"event":"delta","text":"Hel"}

id: 2
data: {"event":"delta","text":"lo"}

id: 3
data: {"event":"done"}
```

To resume after receiving 2 events, repeat the **same request** (same body) with:

```
X-Stream-Id: <same id>
Last-Event-ID: 2
```

The server replays events 3 onwards and then follows the live output. It does not start a new generation.

- Repeating the first request with the same `X-Stream-Id` and no `Last-Event-ID` also attaches to the running stream instead of generating twice.
- `410 Gone` means the stream is unknown, expired, or `Last-Event-ID` is past the end of the log (for example the server restarted). The request has to be restarted from scratch.
- A generation with no client attached is cancelled after 90 seconds. A finished stream stays resumable for 5 minutes. A new generation request cancels unattended ones, because the model runs one generation at a time.
- Requests **without** `X-Stream-Id` behave as before: the generation stops when the connection closes.
- Applies to all streaming endpoints: the generation `*Stream*` routes and `/v1/multimodal/analyzeStream`.
- Browser (Wasm) clients need the server to allow the `X-Stream-Id` and `Last-Event-ID` request headers and expose `X-Stream-Resume` via CORS.

`LlamatikServerClient` does all of this automatically, see [Connecting to a server](../../guides/connecting-to-a-server/#reconnecting-mid-stream).

## Sections

- [Models Status](models/) — loaded models and capabilities for every bridge
- [Generation](generation/) — text generation, streaming, JSON output, KV sessions, MTP, model metadata
- [Embeddings](embeddings/) — embedding vectors
- [Whisper](whisper/) — speech-to-text
- [Stable Diffusion](stablediffusion/) — image and video generation
- [Multimodal](multimodal/) — vision-language streaming
