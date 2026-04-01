# 🏭 OpenClaw Harness Engineering

> Turn your OpenClaw agent into a self-improving engineering factory. Plan → Build (via ACP) → Review → Iterate → Ship.

[![OpenClaw](https://img.shields.io/badge/OpenClaw-compatible-blue?style=flat-square)](https://openclaw.ai)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)

## What is this?

OpenClaw Harness is a **native engineering orchestration framework** for OpenClaw. Your main agent becomes the Lead — it plans sprints, spawns Builder agents via ACP, reviews their output, iterates until quality passes, and ships.

```
You: "Add a favorites feature to my app"

Lead Agent (your OpenClaw agent):
  → Reads codebase, writes SPRINT.md
  → Spawns Claude Code via ACP → code gets written
  → Reviews output (4 dimensions, 1-10 score)
  → If score < 7.0 → sends feedback, Builder iterates
  → Score ≥ 7.0 → commits, pushes, deploys
  → Reports back to you on Telegram/Discord
```

No Docker. No LangGraph. No Python virtualenv. Just OpenClaw + ACP.

## Why not DeerFlow?

| | DeerFlow 2.0 | OpenClaw Harness |
|---|---|---|
| **Setup** | Docker + LangGraph + 30min | `clawhub install harness-factory` |
| **Runtime** | LangGraph Server (custom) | OpenClaw Gateway (already running) |
| **Builder** | CLI wrappers for Claude/Codex | Native ACP sessions |
| **Memory** | Custom memory system | OpenClaw memory_search |
| **Sandbox** | Docker containers | OpenClaw exec |
| **Channels** | Custom IM integrations | Native Telegram/Discord/WhatsApp |
| **Cost** | Separate server needed | Runs on existing OpenClaw |
| **Self-improvement** | ❌ | ✅ Cross-sprint learning (v0.9) |

**TL;DR:** DeerFlow rebuilds everything from scratch. We stand on OpenClaw's shoulders.

## How It Works

### The 5-Phase Pipeline

```
┌─────────┐   ┌─────────┐   ┌─────────┐
│  SCOUT  │──▶│  BUILD  │──▶│ REVIEW  │──┐
│ (Lead)  │   │  (ACP)  │   │  (Lead) │  │
└─────────┘   └─────────┘   └─────────┘  │
                                          │
              ┌─────────┐                 │
              │ ITERATE │◀────────────────┘ if < 7.0
              │  (ACP)  │
              └────┬────┘
                   │
              ┌─────────┐
              │  SHIP   │  if ≥ 7.0
              │ (Lead)  │
              └─────────┘
```

1. **Scout** — Lead reads codebase, writes SPRINT.md with success criteria
2. **Build** — Claude Code (via ACP) implements the sprint
3. **Review** — Lead scores on 4 dimensions (Functionality, Quality, Security, Edge Cases)
4. **Iterate** — If score < 7.0, Lead writes feedback, Builder fixes (max 3 rounds)
5. **Ship** — Commit, push, deploy, report

### 4-Dimension Scoring

| Dimension | Weight | What It Measures |
|-----------|--------|-----------------|
| Functionality | 30% | Does it work as specified? |
| Code Quality | 25% | Clean, DRY, documented? |
| Security | 25% | Auth, CORS, input validation? |
| Edge Cases | 20% | Errors, empty input, timeouts? |

**Security floor rule:** Auth bypass or secret leak = automatic FAIL regardless of other scores.

## Quick Start

### Install
```bash
clawhub install harness-factory
# or
openclaw skills install harness-factory
```

### Manual Install
```bash
git clone https://github.com/guixiang123124/openclaw-harness.git
cp -r openclaw-harness/skills/* ~/.openclaw/skills/
cp -r openclaw-harness/agents/ ~/.openclaw/workspace/openclaw-harness/agents/
cp -r openclaw-harness/templates/ ~/.openclaw/workspace/openclaw-harness/templates/
```

### ACP Setup (for native Builder spawning)
```bash
# Enable ACP backend
openclaw config set plugins.entries.acpx.enabled true
openclaw config set acp.enabled true
openclaw config set acp.backend acpx
openclaw config set acp.defaultAgent claude
openclaw config set plugins.entries.acpx.config.permissionMode approve-all
openclaw gateway restart
```

### Usage

Tell your agent:
```
"Use harness mode to add a password reset endpoint to my API"
```

Or be explicit:
```
"Activate harness engineering for /path/to/project — build a favorites feature"
```

## Sprint Sizing Guide

| Sprint Type | Max Lines | Success Rate | Notes |
|------------|----------|-------------|-------|
| Backend API endpoint | 100 | **95%+** | Include schema + patterns |
| Backend refactor | 150 | **80%+** | Clear before/after spec |
| Frontend component | 100 | **70%+** | ONE component per sprint |
| Frontend page rewrite | 300+ | **<30%** | ⚠️ Always split |
| Full-stack feature | any | **<20%** | ⚠️ Always split backend + frontend |

## Real Examples

### ArkRoute: Password Change Endpoint
- **Score:** 8.55/10 (first-try pass)
- **Builder:** Claude Code via ACP
- **Time:** ~5 minutes total
- **Artifacts:** [examples/arkroute-password-change/](examples/arkroute-password-change/)

### TrendMuse: Dashboard 2.0 (4 sprints)
- **Scores:** 8.55, 8.25, 8.80, 9.00
- **Features:** Weekly Report API, Auto-sync, Dashboard rewrite, Search + Featured
- **Artifacts:** [examples/trendmuse-upgrade/](examples/trendmuse-upgrade/)

## Project Structure

```
openclaw-harness/
├── skills/harness-engineering/
│   └── SKILL.md              ← Core: the executable pipeline
├── agents/
│   ├── builder.md            ← Builder agent system prompt
│   └── reviewer.md           ← Independent Reviewer prompt
├── templates/
│   ├── backend-api.md        ← Sprint template for APIs
│   ├── frontend-component.md ← Sprint template for UI
│   └── fullstack-feature.md  ← Auto-splitting guide
├── examples/                  ← Real sprint artifacts
├── DESIGN-v0.8.md            ← Architecture + DeerFlow analysis
├── ROADMAP.md                ← v0.8 → v1.0 plans
└── README.md                 ← This file
```

## Inspired By

- **[DeerFlow](https://github.com/bytedance/deer-flow)** — ByteDance's LangGraph-based SuperAgent harness. We learned from their middleware architecture and harness/app split.
- **[Meta-Harness](https://arxiv.org/abs/2603.28052)** — Stanford's self-improving harness optimization. We're bringing cross-sprint learning to v0.9.
- **[DSPy](https://github.com/stanfordnlp/dspy)** — Omar Khattab's declarative LM programming. The scoring and iteration philosophy.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
