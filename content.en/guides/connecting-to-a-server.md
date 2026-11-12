---
title: "Connecting to a server"
weight: 16
---

The Llamatik app can use a [Llamatik Server](../../llamatik-server/) for generation instead of (or alongside) a local model. This is useful when the device is too slow or the model too large, or when you want to share one powerful machine between several devices.

> **Heads-up: connections are not encrypted yet.** Messages sent to the server travel over plain HTTP, so they are **not 100% secure** and anyone on the same network could potentially read them. Be careful about what you send, and avoid sharing passwords or sensitive personal information. Encrypted (HTTPS) connections are planned; both encrypted and unencrypted modes will be supported.

## Connect from the app

Open **Settings → Server**:

- **Discover on network** — scan the local network for nodes advertising the `_llamatik._tcp` mDNS service and tap *Connect*. Llamatik Server advertises itself automatically (see [Configuration](../../llamatik-server/configuration/)); if it doesn't show up, add it manually. mDNS needs the phone and server on the same network, and it usually doesn't work across Docker's default bridge network, VPNs or guest Wi-Fi.
- **Add manually** — enter the IP address, port and an optional name, then tap *Connect*.
- **Saved servers** — servers you connected to before can be reconnected with one tap.

Before the first connection to any server the app shows an *unencrypted connection* warning that you must confirm. The same warning is shown as a card at the top of the server screen.

Once connected, the server card lists its **available capabilities** (text, image, vision, video, audio), based on the models currently loaded on the server. The Settings screen also shows `Connected · <name> (<host:port>)` under the **Server** row.

## Chat without a local model

If you skip downloading a model and connect a server, the chat screen shows the normal input box and sends prompts to the server. The *No Model Selected* view only appears when there is **no local model and no connected server**.

Responses are **streamed** (Server-Sent Events), so text appears token by token.

## Status indicator

While a server is connected, the chat toolbar shows a status dot. The app checks the server every 10 seconds:

| Dot | Meaning |
|---|---|
| Green | Reachable and a text generation model is loaded |
| Yellow | Reachable, but no text model is loaded (generation falls back to local) |
| Red | Not reachable |
| Grey | First check still in progress |

The same check also refreshes the capability list, so models loaded on the server after you connected show up without reconnecting.

## Fallback to local generation

If the server is unreachable, has no text model loaded, times out or fails mid-request, the app falls back to local generation when a local model is available. When the request already failed, a dialog offers to retry the same prompt locally.

## Using the client from code

The `:llamatik-network` module provides `LlamatikServerClient`, the same client the app uses:

```kotlin
val client = LlamatikServerClient(LlamatikServerConfig(host = "192.168.1.20", port = 8080))

if (client.isReachable()) {
    val status = client.getModelsStatus()           // what is loaded on the server
    client.streamGenerateWithContext(
        systemPrompt = "You are a helpful assistant.",
        contextBlock = "",
        userPrompt = "Explain Kotlin Multiplatform in one sentence.",
    ).collect { chunk -> print(chunk) }              // streamed token by token
}
```

Notes:

- The default HTTP client sends `Content-Type: application/json`. If you pass your own `HttpClient`, install `ContentNegotiation` with JSON **and** set that default content type, otherwise requests fail with *Fail to prepare request body for sending*.
- Streaming calls must read the response as it arrives (`preparePost(...).execute { }`). A plain `post(...)` buffers the whole body first and defeats streaming. The built-in `stream*` functions already do this.

## Web (Wasm) clients

Browsers enforce CORS. Llamatik Server allows the `Content-Type` header for cross-origin requests; make sure you run a server version that includes it, otherwise the preflight (`OPTIONS`) request is rejected with `403 Forbidden` and the app reports that it cannot connect.

Also note that a page served over HTTPS cannot call a plain `http://` server (mixed content). Until encrypted connections are available, use a server on the same scheme as the page, or a TLS reverse proxy — see [Deployment](../../llamatik-server/deployment/#https-with-a-reverse-proxy).
