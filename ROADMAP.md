# Roadmap — OpenClaw Harness Engineering

## Vision
The first production-grade, one-click installable AI engineering team framework. Turn any single AI agent into a structured engineering team with planning, building, reviewing, and iterating — all orchestrated automatically.

## Current: v0.7 (2026-03-30)
- ✅ 5-Phase workflow (Plan → Build → Evaluate → Iterate → Ship)
- ✅ Claude Code native integration (--agent, worktrees, --bare)
- ✅ 4-dimension scoring system
- ✅ Sprint sizing guide (data-driven from 4 production sprints)
- ✅ 3 real case studies
- ✅ Published on ClawHub

## Next: v0.8 — Sprint Templates Library
- [ ] Model Integration Template (for AI API platforms like ArkRoute)
- [ ] SaaS Feature Template (for products like FashionFlow)
- [ ] Data Pipeline Template (for ETL/scraping like TrendMuse)
- [ ] Frontend Component Template (learned from Sprint 2 failure)
- [ ] Security Audit Template

## v0.9 — Multi-Agent Intelligence
- [ ] Parallel Builder support with automatic task splitting
- [ ] Reviewer independence scoring (prevent evaluation bias)
- [ ] Cross-sprint learning (carry lessons from sprint N to sprint N+1)
- [ ] Sprint auto-planning from GitHub issues

## v1.0 — Harness CLI
- [ ] `npx harness init` — Analyze codebase, generate SPRINT.md
- [ ] `npx harness build` — Run Builder automatically
- [ ] `npx harness review` — Run independent Reviewer
- [ ] `npx harness ship` — Commit + push + deploy
- [ ] GitHub Action for PR auto-review

## Test Beds
These production projects continuously validate and improve the harness:

| Project | Role | Sprint Frequency |
|---------|------|-----------------|
| **TrendMuse** | Full-stack web app | 2-3 sprints/week |
| **FashionFlow** | SaaS product | 1-2 sprints/week |
| **ArkRoute** | API platform | 1 sprint per new model |

Every sprint generates data that feeds back into harness improvements.

## Philosophy
- **Build the tool by using the tool** — We eat our own dogfood
- **Data over opinions** — Sprint sizing guide comes from real success rates, not theory
- **One-click installable** — `clawhub install harness-factory` and you're running
- **Open source** — The community will find patterns we haven't seen yet
