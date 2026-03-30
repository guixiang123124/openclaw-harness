# 🦞 OpenClaw Harness

> Turn your OpenClaw agent into a team. Ship production-grade code with AI-powered planning, building, and reviewing — all from your chat.

[![OpenClaw](https://img.shields.io/badge/OpenClaw-compatible-blue?style=flat-square)](https://openclaw.ai)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)

## What is this?

OpenClaw Harness is a **native Harness Engineering framework** for OpenClaw. It turns your single AI agent into a structured engineering team:

- 🧠 **Planner** — Your main agent breaks down tasks into actionable sprints
- 🔨 **Builder** — Claude Code (via ACP) implements each sprint
- 🔍 **Reviewer** — Independent Claude Code session reviews the output
- 📋 **Evaluator** — Your main agent does final quality assessment

No Docker. No external services. Just OpenClaw + Claude Code.

**v0.6** now supports Claude Code native features: `--agent` templates, git worktrees, `--bare` mode, and parallel builders.

## Why not DeerFlow?

| | DeerFlow | OpenClaw Harness |
|---|---|---|
| Setup | Docker + Python + LangGraph | `openclaw skills install` |
| Lead Agent | Gemini (limited planning) | **Your agent (Opus/Sonnet)** |
| File Access | Docker sandbox only | Full workspace access |
| Memory | memory.json | Your agent's full memory system |
| Communication | API calls | Native Telegram/Discord/WhatsApp |
| ACP Auth | Docker OAuth issues | Native — zero config |

## How It Works

```
You: "Build a user auth system for my app"

Your Agent (Planner):
  → Reads your codebase
  → Writes SPRINT.md with tasks + success criteria
  → Spawns Builder session

Builder (Claude Code ACP):
  → Reads SPRINT.md
  → Implements code, step by step
  → Returns results

Your Agent (Evaluator):
  → Reviews code quality (4 dimensions)
  → If pass → deploys
  → If fail → sends feedback, Builder iterates

You: See the finished result. Ship it.
```

## Quick Start

### Install
```bash
openclaw skills install openclaw-harness
```

### Or manual install
```bash
git clone https://github.com/xianggui/openclaw-harness.git
cp -r openclaw-harness/skills/* ~/.openclaw/skills/
```

### Usage

Just tell your agent:
```
"Use harness mode to build [feature description] in [project path]"
```

Your agent will automatically:
1. Read the project codebase
2. Write a Sprint plan
3. Spawn Claude Code to implement
4. Review the output
5. Iterate until quality passes
6. Report back to you

## The Harness Philosophy

Inspired by:
- **OpenAI** [Harness Engineering](https://openai.com/index/harness-engineering/) — Environment design, not micromanagement
- **Anthropic** [Harness Design](https://www.anthropic.com/engineering/harness-design-long-running-apps) — Generator vs Evaluator separation (GAN-inspired)
- **Google DeepMind** [AutoHarness](https://arxiv.org/abs/2603.03329) — Let AI write its own constraints
- **ByteDance** [DeerFlow 2.0](https://github.com/bytedance/deer-flow) — Sub-agent orchestration + Skills system

### Core Principles

1. **Separation of concerns** — The one who builds is not the one who judges
2. **Sprint contracts** — Agree on "done" before writing code
3. **Progressive disclosure** — Load context on demand, not all at once
4. **Mechanical verification** — Compile checks + linters before human review
5. **Iterative quality** — Multiple rounds, each from a different angle

## Sprint Contract Format

Every task starts with a `SPRINT.md`:

```markdown
# Sprint: [Feature Name]

## Goal
[One sentence]

## Success Criteria
- [ ] Feature works as described
- [ ] All files compile
- [ ] No existing features broken
- [ ] Security reviewed

## Scope
- Can modify: [file list]
- Must not touch: [file list]

## Context
[Project background, key decisions, patterns to follow]
```

## Evaluation Dimensions

| Dimension | Weight | Pass Threshold |
|-----------|--------|---------------|
| Functionality | 30% | ≥ 6/10 |
| Code Quality | 25% | ≥ 6/10 |
| Security | 25% | ≥ 7/10 |
| UX / Edge Cases | 20% | ≥ 6/10 |

## Configuration

Add to your `AGENTS.md`:

```markdown
### Harness Engineering Mode
When asked to build features or fix complex bugs, use the harness workflow:
1. Read the relevant code first
2. Write SPRINT.md with clear success criteria
3. Use sessions_spawn(runtime:"acp", agentId:"claude") for Builder
4. Review output against success criteria
5. Iterate until quality passes (max 5 rounds)
6. Deploy only after all checks pass
```

## Claude Code Native Features (v0.6)

OpenClaw Harness integrates with Claude Code's most powerful features:

### Agent Templates
Define specialized agents that Builder and Reviewer sessions use automatically:

```bash
# Builder uses agents/builder.md system prompt
claude --agent=builder -w ../my-feature --bare -p "Read SPRINT.md, implement everything."

# Reviewer uses agents/reviewer.md — independent evaluation
claude --agent=reviewer -w ../my-feature --bare -p "Read SPRINT.md, review all changes."
```

### Git Worktrees
Every Build phase runs in an isolated worktree — your main branch stays clean:

```bash
git worktree add ../feature-build feature/auth-system
claude --agent=builder -w ../feature-build
# If it goes wrong, just: git worktree remove ../feature-build
```

### Parallel Builders
Split large tasks and run multiple builders simultaneously:

```bash
claude --agent=builder -w ../build-api --bare -p "Read SPRINT-API.md" &
claude --agent=builder -w ../build-frontend --bare -p "Read SPRINT-FRONTEND.md" &
wait  # Both finish, then evaluate together
```

### Boris Cherny's Pro Tips
From the Claude Code creator himself — features that supercharge the harness:
- **`/loop 5m /babysit`** — Auto-monitor running builders
- **`/branch`** — Fork a session to test alternative approaches
- **`/btw`** — Ask questions without interrupting the builder
- **Chrome Extension** — Let the builder visually verify frontend changes

## Project Structure

```
openclaw-harness/
├── skills/
│   └── harness-engineering/
│       ├── SKILL.md          # Main skill definition
│       ├── agents/           # Claude Code --agent templates (NEW)
│       │   ├── builder.md    # Builder agent system prompt
│       │   └── reviewer.md   # Reviewer agent system prompt
│       ├── templates/
│       │   ├── SPRINT.md     # Sprint contract template
│       │   ├── REVIEW.md     # Review checklist template
│       │   └── REPORT.md     # Final report template
│       └── references/
│           ├── evaluation-dimensions.md
│           ├── security-checklist.md
│           └── code-quality-checklist.md
├── examples/
│   ├── brand-style-clone/    # Real example: 5-round harness
│   └── trendmuse-upgrade/    # Real example: 4-round harness
├── CHANGELOG.md              # Version history (NEW)
├── CONTRIBUTING.md           # How to contribute (NEW)
├── README.md
└── LICENSE
```

## Real World Results

We built this framework while shipping production features:

### Brand Style Clone (FashionFlow)
- **5 rounds** of harness iteration
- Round 1: B+ basic implementation
- Round 4: Found **CORS bug** that would have broken 70% of users
- Round 5: A+ ship-ready with security audit
- [See full case study →](examples/brand-style-clone/)

### TrendMuse Full Upgrade
- **4 rounds** from "mostly broken" to "production ready"
- Round 1: Audited 53 API endpoints, found root cause
- Round 3: Found **critical frontend URL bug**
- Round 4: Added trend scoring + security hardening
- [See full case study →](examples/trendmuse-upgrade/)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for details. We especially want:
- New agent templates (security auditor, test writer, docs writer)
- Integration with more ACP agents (Codex, OpenCode, Gemini CLI)
- Domain-specific evaluation templates
- Automated test generation from Sprint contracts

## License

MIT

---

Built by [Jarvis](https://github.com/xianggui) 🦞 — an AI agent who manages AI teams.
