---
title: "Getting Started"
weight: 1
---

# Getting Started with Llamatik Code

## Requirements

- IntelliJ IDEA 2026.1 or any JetBrains IDE built on platform 252 or later
- A compatible GGUF language model file on disk
- macOS, Linux, or Windows with 64-bit JDK 21+

## Installation

1. Open **Settings → Plugins → Marketplace**.
2. Search for **Llamatik Code**.
3. Click **Install**, then restart the IDE.

Alternatively, download the `.zip` from the JetBrains Plugin Marketplace and install via **Settings → Plugins → Install Plugin from Disk**.

## First run

After installation a **Llamatik Code** tool window appears on the right side of the IDE.

1. Open **Settings → Llamatik Code → Inference**.
2. Set the **Model path** to a GGUF file on your disk.
3. Adjust context size and thread count to match your hardware.
4. Return to the tool window and start chatting.

## Choosing a model

Any GGUF model compatible with `llama.cpp` works. Recommended starting points:

- **Fast / lightweight** — 3B–7B quantised models (Q4_K_M or Q5_K_M)
- **Balanced** — 13B–14B quantised models
- **High quality** — 34B+ if your hardware supports it

For agent and tool use, prefer instruction-tuned variants (names ending in `-Instruct` or `-Chat`).

## Ask mode vs Agent mode

The tool window has two primary modes:

- **Ask mode** — Conversational Q&A. The model reads context but does not write files.
- **Agent mode** — Task execution. The agent calls tools, edits files, runs builds, and iterates.

Switch between modes using the mode selector at the top of the tool window.

## Privacy

Inference runs locally by default. No data is sent to Llamatik servers. If you connect an MCP server that calls an external API, that traffic goes through the MCP server process you configured — the plugin itself does not forward your code. You control what connects to the outside world.

## Free vs Pro

**Every feature is available to individual developers for free.** There are no feature gates, no paywalls, and no time limits.

**Pro** comes in two forms:

- **Individual Pro** — an optional donation to support the project. You get the same features as Free. Use the **Support ♥** button in the plugin header or visit [GitHub Sponsors](https://github.com/sponsors/ferranpons).
- **Organization Pro** — required for companies and teams. Purchasing a seat on the [JetBrains Marketplace](https://plugins.jetbrains.com/plugin/31304) is the correct way to use Llamatik Code commercially and funds continued development ($15/month or $150/year per seat).

See [llamatik.com](https://www.llamatik.com) for details.
