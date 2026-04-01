# Roadmap — OpenClaw Harness Engineering

## Vision
The first production-grade, zero-config AI engineering factory for OpenClaw. DeerFlow's orchestration intelligence + Meta-Harness's self-improving loops + OpenClaw's native ACP ecosystem.

**One sentence:** Turn `clawhub install harness-factory` into a full engineering team that gets better with every sprint.

## Current: v0.8 (2026-03-31)
- ✅ 5-Phase workflow (Scout → Build → Review → Iterate → Ship)
- ✅ SKILL.md rewritten as executable pipeline (not tutorial)
- ✅ Agent templates (builder.md, reviewer.md)
- ✅ Sprint templates (backend-api, frontend-component, fullstack-feature)
- ✅ 4-dimension scoring system with Security floor rule
- ✅ Sprint sizing guide (data from 8+ production sprints)
- ✅ First real ACP sprint: ArkRoute password change (8.55/10)
- ✅ DeerFlow architecture analysis + differentiation
- ✅ ACP configured (acpx backend, claude/codex/pi agents)
- ✅ ACP spawn verified: `sessions_spawn(runtime:"acp")` → accepted

## Next: v0.9 — Self-Improving Pipeline (inspired by Meta-Harness)

**Key insight from Stanford's Meta-Harness paper (2603.28052):**
The optimizer that sees full execution traces (not compressed scores) wins. 
Same principle applies: our Builder that reads prior sprint artifacts outperforms one that starts fresh.

### Cross-Sprint Learning
- [ ] `experience/` directory: SPRINT.md + BUILDER_REPORT.md + REVIEW.md + HARNESS_REPORT.md from every past sprint
- [ ] Builder prompt includes relevant past experiences (grep for similar task type)
- [ ] Anti-patterns auto-extracted from REVIEW.md files (what went wrong)
- [ ] Success patterns auto-extracted from high-scoring sprints (what worked)

### ACP Native Integration
- [ ] `sessions_spawn(runtime:"acp", agentId:"claude")` as primary Builder path
- [ ] Parallel Builders for split sprints (backend + frontend simultaneously)
- [ ] Independent Reviewer via separate ACP session
- [ ] `sessions_yield` for async wait + auto-notify on completion
- [ ] `resumeSessionId` for multi-round iteration without losing context

### Sprint Auto-Planning
- [ ] Lead Agent reads codebase + user request → auto-generates SPRINT.md
- [ ] Auto-detect sprint type from file extensions and patterns
- [ ] Auto-populate "Existing Patterns" from codebase analysis
- [ ] Auto-generate DO NOT BREAK list from existing routes/tests

## v1.0 — Harness CLI + GitHub Integration
- [ ] `npx harness init` — Analyze codebase, generate SPRINT.md
- [ ] `npx harness build` — Run Builder automatically via ACP
- [ ] `npx harness review` — Run independent Reviewer
- [ ] `npx harness ship` — Commit + push + deploy
- [ ] GitHub Action for PR auto-review
- [ ] GitHub Issues → auto-Sprint generation

## v1.1 — Meta-Harness Loop
- [ ] Automated harness-code evolution (the harness improves itself)
- [ ] A/B testing of different Builder prompts
- [ ] Score regression detection (alert if sprint quality drops)
- [ ] Community-shared sprint templates via ClawHub

## Architecture Comparison

```
DeerFlow 2.0:
  LangGraph + Docker + custom memory/sandbox/channels
  = Full agent runtime (heavy, 30min setup)

OpenClaw Harness v0.8:
  OpenClaw ACP + native memory/exec/channels
  = Orchestration layer only (light, one-line install)

Meta-Harness (Stanford):
  Claude Code + filesystem of all prior traces
  = Self-improving optimization loop

OpenClaw Harness v0.9 (our target):
  ACP orchestration + cross-sprint learning + auto-planning
  = DeerFlow's power + Meta-Harness's evolution + OpenClaw's simplicity
```

## Test Beds

| Project | Role | Sprint Type | ACP Agent |
|---------|------|-------------|-----------|
| **ArkRoute** | API platform | backend-api | claude |
| **TrendMuse** | Full-stack web app | fullstack | claude |
| **FashionFlow** | SaaS product | frontend + backend | claude |

Every sprint generates artifacts that feed back into harness improvements.

## Philosophy
- **Build the tool by using the tool** — dogfooding on real products
- **Data over opinions** — sprint sizing comes from real success rates
- **Zero-config** — `clawhub install` and you're running
- **Self-improving** — every sprint makes the next sprint better (Meta-Harness principle)
- **Open source** — community finds patterns we haven't seen
