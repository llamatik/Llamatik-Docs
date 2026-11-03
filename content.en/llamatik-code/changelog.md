---
title: "Changelog"
weight: 7
---

# Changelog

## 1.8.0 — 2026-06-10

### Autonomous Local Development
- **Offline Enterprise Pair Programmer** — structured mission workflow: gather context → classify risks → plan → patch → verify → audit report. Inference runs locally by default; data leaves the machine only through services you explicitly configure.
- **Mission classification** — 11 mission types auto-detected from plain-language requests (CODE_REVIEW, REFACTOR, BUG_FIX, FEATURE_IMPLEMENTATION, TEST_GENERATION, DOCUMENTATION, BUILD_FAILURE_REPAIR, ARCHITECTURE_EXPLANATION, DEPENDENCY_UPDATE, SECURITY_AUDIT, PERFORMANCE_ANALYSIS).
- **Safety levels** — APPROVAL_REQUIRED (default), PLAN_ONLY, AUTO_APPLY, FULL_AUTO.
- **Risk detection** — LOW / MEDIUM / HIGH / CRITICAL risks surfaced in the plan before any file is written. CRITICAL risks require explicit acknowledgement.
- **Plan-before-write enforcement** — agent must produce a mission plan before any write tool call (default: on).
- **Diff-before-write enforcement** — unified diff shown before every patch (default: on).
- **Audit trail** — per-mission metadata log (no raw source code). Accessible via `enterprise_mission_audit`.
- **Rollback instructions** — included in every final audit report.
- 4 new agent tools: `enterprise_mission_create`, `enterprise_mission_context`, `enterprise_mission_audit`, `enterprise_mission_report`.
- 7 new context menu actions under **Llamatik Code → Autonomous Local Development**: Enterprise Pair Programmer, Review Current Changes, Fix Build Failure, Generate Tests, Explain Architecture, Safe Refactor, Implement Feature.
- New settings page: **Settings → Llamatik Code → Autonomous Local Development**.

### Inline completions (opt-in MVP)
- Local fill-in-the-middle completions using the same on-device model as the agent.
- Debounced requests (600 ms default) — stale requests cancelled automatically.
- Current-file only; configurable context window (default 1200 chars) and max tokens (default 64).
- Disabled by default — enable at **Settings → Llamatik Code → Autonomous Local Development → Enable inline completions**.

### Rating & Feedback
- After a configurable number of successful file-change tasks, the plugin shows a non-modal rating prompt.
- Respects a cooldown; dismissible permanently with "Don't ask again".

### File selection picker
- Model file picker (load GGUF from disk) now uses IntelliJ's native `FileChooser` API. Correct macOS permissions, IDE-native look and feel, reliable `.gguf` filtering on all platforms.

### Library
- Updated to Llamatik library 1.8.0 (llama.cpp b9574) with improved KV cache management and Qwen3.6 Mamba-hybrid architecture support.

### Agent loop reliability
- **Attached file context respected** — system prompt now explicitly states that attached file contents are already provided. Model no longer wastes rounds calling `read_file` or `search_code` on content it already has.
- **Cross-round read-only dedup** — `read_file`, `search_code`, and `list_files` calls are tracked across all rounds. Duplicate calls return a synthetic result immediately without consuming a round. Attached file paths are pre-seeded into the dedup set.
- **Write pass forced on all-duplicate rounds** — if every tool call in a round is synthetic, the agent escalates directly to a final write pass (thinking suppressed).
- **Write pass forced on mid-loop prose** — if the model writes plain text instead of a tool call after reading files, the agent escalates to a final write pass.
- **Thinking suppressed on final round** — the model's full token budget goes to the `write_file` output.

### Tag leak fixes
- Orphan `</think>` no longer appears in displayed responses after echo-stripping removes the opening tag.
- Orphan `</arg>`, `</tool_call>`, and `</edit_proposal>` closing tags no longer appear in displayed responses.

---

## 1.5.0 — 2026-05-08

### Code Health Monitor
- New opt-in background analysis layer for proactive code quality monitoring.
- Static checks without a model: large files (>300 lines), TODO/FIXME accumulation (≥3), empty catch blocks, deprecated API usage.
- Regression risk analyser for git diffs: removed public APIs (HIGH), deleted tests (HIGH), Gradle dependency changes, removed error handling, AndroidManifest modifications.
- Before-commit assistant: summarises staged changes and flags regression risks.
- Watched areas: pin files, directories, packages, or modules for priority analysis (Free: 1, Pro: 20).
- Refactor opportunity detection: long functions (>40 lines), repeated catch patterns.
- Session context tracking: recent files, current branch, last build failure.
- 14 new agent tools for all Code Health operations.
- New settings page: **Settings → Llamatik Code → Code Health Monitor**.
- Free: LIGHT mode, manual trigger only. Pro: BALANCED (30 files/30s) and AGGRESSIVE (150 files/120s) modes with background scheduling.

### External Knowledge
- Agent can search the web, read docs pages, and research errors from inside the IDE.
- Research orchestrator: detects intent (library docs, bug, API, migration, version) and plans multi-step searches.
- Smart source routing to kotlinlang.org, developer.android.com, developer.apple.com, spring.io, github.com based on query context.
- Error research: detect build tool and framework; produce structured fix plans.
- Private docs indexing: add up to 10 (Pro) or 1 (Free) custom documentation sources.
- Citations with every external research result (source type, confidence, URL).
- 16 new agent tools for all External Knowledge operations.
- New editor action: **Search Docs & Web** in the right-click context menu.
- New settings page: **Settings → Llamatik Code → External Knowledge**.
- Free: 5 searches/day, 3 pages/task. Pro: 50 searches/day, 10 pages/task.

---

## 1.4.0 — 2026-05-07

### MCP Server Support
- Llamatik Code is now an IDE-native MCP host.
- New settings page: **Settings → Llamatik Code → MCP Servers**.
- Full server lifecycle: add, edit, remove, enable/disable, test connection, view logs, refresh tools.
- 6 new built-in agent tools: `mcp_list_servers`, `mcp_list_tools`, `mcp_call_tool`, `mcp_list_resources`, `mcp_read_resource`, `mcp_list_prompts`.
- Discovered MCP tools bridged as `mcp_<server>_<tool>` first-class agent tools.
- Approval dialog with LOW / MEDIUM / HIGH risk level on every MCP tool call.
- HIGH-risk tools never auto-approved.
- Free: 1 active server, manual approval. Pro: unlimited servers, per-tool and per-server auto-approval.
- Secrets redacted from all log and trace output.

---

## 1.3.0 — 2026-05-06

### Agent tools
- Git: `git_diff`, `git_status`, `git_log`, `git_blame`, `git_changed_files`.
- Project index: `index_search_files`, `index_search_symbols`, `index_list_recent`.
- PSI: `psi_find_usages`, `psi_get_type`, `psi_get_declarations`.
- Codebase rules: `rules_get`, `rules_set`, `rules_clear`.
- Run configs: `run_config_list`, `run_config_execute`.
- KMP native: `kmp_diagnose`, `kmp_fix_plan`.

### Aggressive Mode (Pro)
- `AGGRESSIVE` execution mode for decisive agent runs.
- Configurable fallback policy: ABORT / RETRY / ESCALATE.

### Model Intelligence
- `ModelCapabilityService`: detects tool use, long context, code, and instruction-following capability.
- `ModelBenchmarkService`: measures tokens/second and first-token latency.
- `SmartSearchService`: fuzzy, recency-ranked file and symbol search.

### Snapshots & Context Recovery
- `ProjectSnapshotService`: lightweight project state snapshots before agent runs.
- `ContextRecoveryService`: detects context drift and offers recovery options.

---

## 1.2.0 — 2026-05-04

### Core agent
- Tool-driven agent loop with two LLM passes (plan + execute).
- Safe diff preview before any file write.
- Build verification after apply.
- Agent trace panel with tool calls, duration, and results.

### Multi-task agent (Pro)
- Parallel task execution with a live task queue panel.
- Task cancellation and live status tracking.

### Chat sessions
- Persistent multi-session chat store with per-project isolation.
- Chat session list panel with last-message preview.
- Parallel Ask-mode conversations.

### Context actions
- Editor and project-view right-click actions: Ask About Selection, Explain This File, Refactor Selection, Fix Selected Code, Generate Tests, Add Documentation, Edit with Prompt, Explain Failing Build, Run Diagnostics, Explain Current Git Changes.
