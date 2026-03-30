# Installation Guide

## Method 1: Copy to skills directory (Recommended)
```bash
git clone https://github.com/guixiang123124/openclaw-harness.git
cp -r openclaw-harness/skills/harness-engineering ~/.openclaw/skills/
```

## Method 2: Workspace skills
```bash
cd ~/.openclaw/workspace
git clone https://github.com/guixiang123124/openclaw-harness.git
# Skills in workspace/skills/ are auto-detected
```

## Method 3: ClawHub (coming soon)
```bash
openclaw skills install openclaw-harness
```

## After Installation

Add to your AGENTS.md:
```markdown
### 🏭 Harness Engineering
When task involves: new feature, complex bug fix, refactoring, multi-file changes
→ Read `~/.openclaw/skills/harness-engineering/SKILL.md` and follow the 5-phase workflow.
```

## Verify
Your agent should now see "harness-engineering" in available skills.
Test: ask your agent "Use harness mode to [any task]"

## Requirements
- OpenClaw v2026.2.9+
- Claude Code installed (for ACP Builder sessions)
- `claude auth login` completed (OAuth active)
