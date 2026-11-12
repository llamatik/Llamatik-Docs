<p align="center">
  <img src="https://raw.githubusercontent.com/llamatik/Llamatik/main/assets/llamatik-new-logo.png" alt="Llamatik Logo" width="150"/>
</p>

<h1 align="center">Llamatik Docs</h1>

<p align="center">
  <b>The official documentation for Llamatik, Llamatik Server and Llamatik Code.</b>
</p>

<p align="center">
  Guides · API reference · Self-hosting · IDE plugin
</p>

<p align="center">
  <a href="https://docs.llamatik.com"><img src="https://img.shields.io/badge/Docs-docs.llamatik.com-blue" alt="docs badge"/></a>
  <img src="https://img.shields.io/badge/Built%20with-Hugo-ff4088" alt="hugo badge"/>
  <img src="https://img.shields.io/badge/Theme-hugo--book-green" alt="theme badge"/>
  <img src="https://img.shields.io/badge/Kotlin-Multiplatform-blueviolet" alt="kmp badge"/>
  <img src="https://img.shields.io/badge/LLM-llama.cpp-orange" alt="llama.cpp badge"/>
  <img src="https://img.shields.io/badge/STT-whisper.cpp-blue" alt="whisper.cpp badge"/>
  <img src="https://img.shields.io/badge/Image-stablediffusion.cpp-purple" alt="stablediffusion badge"/>
  <img src="https://img.shields.io/badge/License-MIT-lightgrey" alt="license badge"/>
</p>

---

## ✨ What is this repository?

This repository contains the source of **[docs.llamatik.com](https://docs.llamatik.com)**, the documentation site for the Llamatik ecosystem:

- 🧩 **[Llamatik](https://github.com/llamatik/Llamatik)** — Kotlin Multiplatform AI SDK for on-device LLM, speech-to-text, image/video generation and agents
- 🖥 **[Llamatik Server](https://github.com/llamatik/Llamatik-Server)** — self-hosted HTTP backend with full parity with the Llamatik library API
- 💻 **[Llamatik Code](https://github.com/llamatik/Llamatik-Code)** — local-first AI pair programmer for JetBrains IDEs

The site is built with [Hugo](https://gohugo.io/) and the [hugo-book](https://github.com/alex-shpak/hugo-book) theme.

---

## 📚 What's covered

| Section | Description | Path |
|---|---|---|
| 🚀 **Getting Started** | Installation, quickstart and models | `content.en/getting-started` |
| 🧑‍💻 **Guides** | Text generation, streaming, embeddings, KV cache, Whisper, Stable Diffusion, multimodal, Agent Framework & Platform | `content.en/guides` |
| 📖 **API Reference** | `LlamaBridge`, `WhisperBridge`, `StableDiffusionBridge`, `MultimodalBridge`, `GenStream`, `LlamaSession` | `content.en/api` |
| 🖥 **Llamatik Server** | Getting started, configuration, deployment | `content.en/llamatik-server` |
| 💻 **Llamatik Code** | Agent & tools, MCP servers, autonomous development, settings, changelog | `content.en/llamatik-code` |
| 📋 **Migration** | Upgrading from 1.x to 2.0 | `content.en/migration` |
| 🛠 **Troubleshooting** | Android native and iOS linking pitfalls | `content.en/troubleshooting` |
| 🕰 **Legacy** | v1.x documentation | `content.en/v1-legacy` |

---

## 🔧 Requirements

- [Hugo](https://gohugo.io/installation/) **extended** edition
- Git (the theme is a submodule)

---

## ⚡ Quick Start

```bash
# Clone with the theme submodule
git clone --recurse-submodules https://github.com/llamatik/Llamatik-Docs.git
cd Llamatik-Docs

# Already cloned without submodules?
git submodule update --init --recursive

# Start the local dev server (http://localhost:1313)
hugo server
```

Build the static site into `public/`:

```bash
hugo --minify
```

---

## 🧱 Repository structure

```
Llamatik-Docs
├── content.en/     Documentation pages (Markdown)
├── assets/         Site assets
├── static/         Static files (logo, favicon)
├── layouts/        Template overrides
├── i18n/           Translation strings
├── themes/         hugo-book theme (git submodule)
├── archetypes/     Content templates
└── hugo.toml       Site configuration
```

---

## 🤝 Contributing

Documentation improvements are very welcome:

- Fix typos and unclear explanations
- Add examples and guides
- Keep pages in sync with new Llamatik, Server and Code releases
- Add or improve translations

Create or edit pages under `content.en/` (each folder uses an `_index.md` for its section), preview with `hugo server`, then open an issue or PR 🚀

---

## 📦 Related Projects

- 🔗 **Llamatik Library** — https://github.com/llamatik/Llamatik
- 🔗 **Llamatik Server** — https://github.com/llamatik/Llamatik-Server
- 🔗 **Llamatik Code** — https://github.com/llamatik/Llamatik-Code
- 🌐 **Website** — https://www.llamatik.com

---

## 📜 License

This project uses a dual-license model:

| Component | License |
|---|---|
| Documentation source (Markdown, Hugo config, layouts) | [MIT License](./LICENSE) |
| Visual assets — logos, icons, favicons, screenshots | [Proprietary](./LICENSE-assets) |

The documentation source is freely usable under the MIT License. The **Llamatik name and
logo are proprietary marks** owned by Ferran Pons and are not covered by the MIT
License. See [LICENSE-assets](./LICENSE-assets) for the full terms that apply to
all graphic files under `static/` and `assets/`.

The [hugo-book](https://github.com/alex-shpak/hugo-book) theme is a git submodule and keeps its own MIT license.

---

Built with ❤️ for the Kotlin & AI community.
