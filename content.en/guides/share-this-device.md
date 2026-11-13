---
title: "Share this device"
weight: 17
---

The Llamatik app can act as a server for your other devices. Turn on **Settings → Share this device** and any other Llamatik app on the same network can send prompts to this device and use the models you already have loaded, the same way it would use a [Llamatik Server](../../llamatik-server/). Use it to let a phone borrow your desktop's GPU, or to share one big model between several devices.

Sharing is **off by default** and is never started automatically: you turn it on each time.

## Supported platforms

| Platform | Hosting |
|---|---|
| Android | Yes. Runs in a foreground service with wake and Wi-Fi locks, so it keeps serving while the app is in the background. |
| Desktop (JVM) | Yes |
| iOS | Yes, **foreground only**. iOS suspends apps that leave the foreground, so the app keeps the screen awake while sharing. The host is advertised over Bonjour. |
| Web (Wasm) | No. A browser cannot open a listening socket. The option is hidden. |

## What is shared

- **Text generation** and **embeddings**, using the models currently loaded in the app. Loading, unloading and reconfiguring models stays on the host; remote clients cannot call admin endpoints (they get `403`).
- If you are generating in the app yourself, remote requests get `503 Retry-After` instead of interleaving with your chat.

Other modalities (image, vision, audio, video) are present in the server library but are not exposed by the app host yet.

## Turn it on

1. Load a model in the app (the **Models** tab).
2. Open **Settings → Share this device** and switch **Share this device** on. You can change the port (default `8080`).
3. The screen shows the address other devices can connect to, the models being shared and the number of requests served.

## Approving devices

Every device must be approved by you before it can use yours:

1. On the other device, open **Settings → Connect to Server** and connect to this device (discover it on the network or enter its address; see [Connecting to a server](../connecting-to-a-server/)).
2. This device shows an **Allow this device?** prompt with the other device's name. Tap **Allow** or **Deny**.
3. Approved devices are listed under **Approved devices** and persist across restarts. Tap **Revoke** to remove one.

If the request is not approved in time, the other device shows *The request was not approved in time*.

> **Heads-up: traffic between devices is not encrypted.** Only approve devices you trust, on a network you trust. Generating for others also uses your battery and may heat the device.

## Seeing who is connected

While sharing is on, the app shows the host status in two places:

- **Settings** shows a line under *Share this device*:
  - `Sharing · no devices connected`
  - `Sharing · 2 connected`
  - `Sharing · 2 connected · generating for them`
- **Chat top bar** shows a small share icon next to the server status dot:

| Icon | Meaning |
|---|---|
| Grey | Sharing, but nobody is connected |
| Green with a number | That many devices are connected |
| Pulsing orange | Generating for another device right now |

A device counts as **connected** if it made a request in the last 30 seconds. Connected clients check in about every 10 seconds, so a device that closes the app or loses the network drops out after roughly 30 seconds. The status refreshes every 5 seconds.

## For developers

The embedded server is built from two modules in the Llamatik repository:

- **`:server-lib`** (published as `com.llamatik:server`): the engine-agnostic Ktor inference API shared with the standalone Llamatik Server. Entry point: `installLlamatikServer(LlamatikServerConfig)`. It includes per-device approval (`PairingManager`, `/v1/pairing/...`).
- **`:host`**: `EmbeddedServer` on Ktor CIO, plus the Android foreground service and the iOS/desktop host controllers.

The server shares the app's `LlamaBridge` singleton, so remote clients always use the models already loaded in the app.
