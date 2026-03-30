# Changelog

## v0.6 — Claude Code Native Integration (2026-03-30)

### Added
- **Claude Code agent templates** — `.claude/agents/builder.md` and `.claude/agents/reviewer.md` for native `claude --agent` support
- **Worktree workflow** — Builder operates in isolated git worktree (`claude -w`), protecting main branch
- **Multi-agent support** — Framework now supports Claude Code, Codex, and Gemini CLI as Builder agents
- **`--bare` mode** — SDK calls use `--bare` for 10x faster startup in non-interactive mode
- **CONTRIBUTING.md** — Guidelines for community contributors
- **CHANGELOG.md** — Version history tracking

### Changed
- SKILL.md Phase 2 (BUILD) updated with worktree and `--agent` workflows
- README updated with Claude Code native features section
- Evaluation dimensions expanded with real-world calibration notes

### Inspiration
- Boris Cherny (Claude Code creator) — 15 essential CC features (2026-03-29)
- Our own production experience: Brand Style Clone (5 rounds), TrendMuse (4 rounds)

## v0.5 — Production Verification (2026-03-29)
- Install guide with 3 methods
- Metrics tracking system
- Production-tested with TrendMuse migration

## v0.4 — Metrics System (2026-03-29)
- Evaluation metrics + AutoResearch feedback loop
- Initial tracking data from real deployments

## v0.3 — Safety & Quality (2026-03-28)
- Security/quality checklists
- Report template
- Real case studies: Brand Style Clone + TrendMuse

## v0.2 — Core Workflow (2026-03-28)
- Complete 5-phase SKILL.md
- 4-dimension scoring system
- Anti-pattern guide

## v0.1 — Initial Release (2026-03-28)
- Project skeleton + README
