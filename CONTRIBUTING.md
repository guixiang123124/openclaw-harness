# Contributing to OpenClaw Harness

Thanks for your interest! Here's how to help.

## What We Need Most

1. **New agent templates** — `.claude/agents/` definitions for specialized roles (security auditor, test writer, docs writer)
2. **Multi-agent support** — Integration with Codex, Gemini CLI, OpenCode, or other ACP-compatible agents
3. **Evaluation templates** — Domain-specific scoring (mobile apps, APIs, data pipelines, ML models)
4. **Real case studies** — `examples/` showing the harness workflow in your projects
5. **Automated testing** — Generate tests from Sprint success criteria

## How to Contribute

1. Fork the repo
2. Create a branch: `git checkout -b feature/your-feature`
3. Make your changes
4. Test with a real project (dogfood it!)
5. Submit a PR with:
   - What you changed and why
   - A real example of the feature working
   - Any SKILL.md changes

## Code Style

- Markdown files: 80-char soft wrap
- YAML frontmatter: follow existing SKILL.md format
- Templates: use `[PLACEHOLDER]` for user-filled values
- Keep it practical — every feature should have a "when to use" explanation

## Skill Structure

```
skills/harness-engineering/
├── SKILL.md                    # Main skill (the agent reads this)
├── agents/                     # Claude Code --agent templates
│   ├── builder.md
│   └── reviewer.md
├── templates/                  # User-facing templates
│   ├── SPRINT.md
│   ├── REVIEW.md
│   └── REPORT.md
└── references/                 # Supporting docs (agent reads on demand)
    ├── evaluation-dimensions.md
    ├── security-checklist.md
    └── code-quality-checklist.md
```

## Questions?

Open an issue or find us on [OpenClaw Discord](https://discord.com/invite/clawd).
