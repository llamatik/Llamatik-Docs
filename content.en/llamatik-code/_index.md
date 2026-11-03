---
title: "Llamatik Code"
weight: 50
bookCollapseSection: true
---

# Llamatik Code

**Llamatik Code** is a local-first AI coding assistant built for **Kotlin Multiplatform development** and every language in your JetBrains IDE. It runs models locally by default — local by default, distributed by choice — and ships with dedicated KMP tools for diagnosing native compilation errors, planning cross-platform refactors, and routing documentation searches to the right platform references automatically.

## Quick links

- **[Getting Started](getting-started/)** — install the plugin, point it at a model, start chatting
- **[Agent & Tools](agent-and-tools/)** — how the agent loop works and what tools are available
- **[Autonomous Local Development](autonomous-local-development/)** — mission-driven coding: plan → patch → verify → audit (1.8.0+)
- **[Offline Enterprise Pair Programmer](offline-enterprise-pair-programmer/)** — deep dive into the mission workflow (1.8.0+)
- **[Inline Completions](inline-completions/)** — passive AI suggestions as you type (1.8.0+)
- **[Trust and Safety](trust-and-safety/)** — plan enforcement, diff gates, audit trail, and privacy guarantees
- **[MCP Servers](mcp-servers/)** — connect any MCP-compatible tool server to the agent (1.4.0+)
- **[External Knowledge](external-knowledge/)** — web search, docs research, and error analysis (1.5.0+)
- **[Code Health Monitor](code-health-monitor/)** — background code quality analysis (1.5.0+)
- **[Settings Reference](settings-reference/)** — full configuration reference
- **[Changelog](changelog/)** — version history from 1.2.0 to 1.8.0

## Key features

- **Kotlin Multiplatform focus** — Dedicated `kmp_diagnose` and `kmp_fix_plan` tools; smart routing to KMP, Android, and Apple documentation; awareness of shared/platform source sets
- **Project-aware chat** — Ask questions about your codebase with persistent, multi-session support
- **Local AI models** — Run models locally by default via GGUF; inference stays on your machine unless you explicitly connect external services
- **AI agent for code editing** — Generate and modify code using natural language with safe diff previews
- **Context-aware assistance** — Understands the current file, selection, git state, and project structure
- **Autonomous Local Development** — Mission-driven coding: plan → patch → verify → audit, entirely on-device (1.8.0+)
- **Inline completions** — Passive fill-in-the-middle suggestions as you type (1.8.0+)
- **MCP server support** — Connect any MCP-compatible tool server (1.4.0+)
- **External Knowledge** — Research the web, official docs, and GitHub issues from inside the IDE (1.5.0+)
- **Code Health Monitor** — Proactive background analysis for regressions, complexity, and refactor opportunities (1.5.0+)
- **Multi-task agent** — Run multiple agent tasks in parallel with a live task queue panel (Pro)
- **Aggressive mode** — Let the agent act decisively without prompting on each step (Pro)

## Free vs Pro

**Every feature is free for individual developers.** There are no feature gates or paywalls.

**Pro** exists in two forms:

- **Individual Pro** — a voluntary way to support the project. No extra features; same as Free. Available via the **Support ♥** button in the plugin or [GitHub Sponsors](https://github.com/sponsors/ferranpons).
- **Organization Pro** — required for companies and teams using Llamatik Code commercially. Available on the [JetBrains Marketplace](https://plugins.jetbrains.com/plugin/31304). Funds maintenance, new features, and model catalog updates.

See [llamatik.com](https://www.llamatik.com) for details.
