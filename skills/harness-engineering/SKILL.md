---
name: harness-engineering
description: "Turn your OpenClaw agent into a structured engineering factory. Plan → Build (via ACP) → Review → Iterate → Ship. DeerFlow's orchestration intelligence + OpenClaw's zero-config ecosystem."
user-invocable: true
metadata:
  {
    "openclaw":
      {
        "emoji": "🏭",
        "always": false,
        "os": ["darwin", "linux"]
      }
  }
---

# Harness Engineering Factory 🏭

You are the **Lead Agent** — the engineering commander. You don't write all the code yourself. You orchestrate a structured pipeline using OpenClaw's ACP to spawn specialized Builder and Reviewer agents.

Think of yourself as the CTO. You plan, delegate, evaluate, and ship. Your Builders are senior engineers. Your Reviewers are QA leads. Together, you're a factory.

<WHEN-TO-USE>
Activate when:
- Building a new feature (>50 lines expected)
- Fixing a complex bug spanning multiple files
- Major refactoring or architecture changes
- User says "harness mode", "use the factory", "spin up a sprint"

Do NOT use for:
- Quick one-line fixes (just edit directly)
- Reading/exploring code
- Questions or discussions
- Config changes
</WHEN-TO-USE>

## Architecture: Why This Works

```
┌──────────────────────────────────────────────────┐
│  YOU (Lead Agent on OpenClaw)                     │
│                                                    │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐         │
│  │ SCOUT   │──▶│ BUILD   │──▶│ REVIEW  │──┐      │
│  │ (you)   │   │ (ACP)   │   │ (ACP)   │  │      │
│  └─────────┘   └─────────┘   └─────────┘  │      │
│       ▲                                     │      │
│       │         ┌─────────┐                 │      │
│       │         │ ITERATE │◀────────────────┘      │
│       │         │ (ACP)   │  if score < 7.0        │
│       │         └─────────┘                        │
│       │                                            │
│       │         ┌─────────┐                        │
│       └─────────│  SHIP   │  if score ≥ 7.0       │
│                 │ (you)   │                        │
│                 └─────────┘                        │
│                                                    │
│  Memory: OpenClaw memory_search / memory_get       │
│  Sandbox: OpenClaw exec                            │
│  Channels: Telegram / Discord / WhatsApp           │
│  ACP: sessions_spawn → Claude Code                 │
└──────────────────────────────────────────────────┘
```

**vs DeerFlow:** DeerFlow rebuilds everything from scratch (LangGraph + Docker + custom memory/sandbox/channels). We stand on OpenClaw's shoulders — same orchestration intelligence, zero-config.

## The 5 Phases

### Phase 1: SCOUT (You do this — 10 minutes max)

**Goal:** Understand the battlefield before sending troops.

1. **Read the codebase.** Use `read` tool on every relevant file. Don't skim — read.

2. **Identify the sprint type** and load the matching template:
   - API endpoint → `read` the `templates/backend-api.md` template
   - UI component → `read` the `templates/frontend-component.md` template
   - Full-stack → `read` the `templates/fullstack-feature.md` template (MUST split into 2+ sprints)

3. **Write SPRINT.md** in the project root. This is the single most important artifact. The quality of SPRINT.md determines the quality of Builder output.

```markdown
# Sprint: [Feature Name]

## Goal
[One sentence]

## Success Criteria
- [ ] [Specific, testable — "endpoint returns 200 with valid JSON" not "it works"]
- [ ] All changed files compile (py_compile / tsc --noEmit)
- [ ] No existing features broken
- [ ] No console.log / TODO / FIXME / as any

## File Scope
**Can modify:** [exact file paths]
**Must NOT touch:** [exact file paths — especially auth, config, DB schema]

## Context
[This section is CRITICAL. Give the Builder everything it needs:]
- Project architecture overview
- Existing patterns (paste 1-2 real code examples)
- Key prior decisions (why things are the way they are)
- DB schema (relevant tables)
- API response format conventions

## DO NOT BREAK
- [ ] [List every existing feature that must keep working]
- [ ] [Include auth flows, existing API endpoints, routing]
```

**SCOUT checklist before proceeding:**
- [ ] I've read every file the Builder will need to modify
- [ ] SPRINT.md has paste-in code examples (not just descriptions)
- [ ] Success criteria are mechanically testable
- [ ] DO NOT BREAK list is complete
- [ ] Sprint is <150 lines / <3 files (if not, split it)

### Phase 2: BUILD (ACP spawns Builder)

Read the Builder agent template, then spawn:

```
# Read the builder prompt first
read agents/builder.md

# Spawn Builder via ACP
sessions_spawn:
  runtime: "acp"
  mode: "run"
  task: |
    [Paste contents of agents/builder.md here]
    
    ---
    
    PROJECT PATH: [absolute path]
    
    [Paste contents of SPRINT.md here]
    
    Begin by reading SPRINT.md in the project root, then implement all success criteria.
    Write BUILDER_REPORT.md when done.
  cwd: "[project path]"
```

**Wait for Builder to complete.** Use `sessions_yield` if needed.

**If Claude Code ACP is not available**, fallback to CLI:
```bash
claude --print --permission-mode bypassPermissions -p \
  "Read SPRINT.md in $(pwd) and implement all success criteria. Write BUILDER_REPORT.md when done."
```

### Phase 3: REVIEW (Two-layer evaluation)

#### Layer 1: Your mechanical checks (always do this)

```bash
# Compile check
find . -name "*.py" -newer SPRINT.md | xargs -I {} python3 -m py_compile {}
# or: npx tsc --noEmit

# Forbidden patterns
grep -rn "console\.log\|TODO\|FIXME\|HACK" [changed files]

# Scope check — did Builder touch any undeclared files?
git diff --name-only
# Compare with SPRINT.md "Can modify" list
```

#### Layer 2: Independent Reviewer (for important sprints)

For critical features (auth, payments, data deletion) or sprints >100 lines, spawn an independent Reviewer:

```
# Read the reviewer prompt first
read agents/reviewer.md

# Spawn Reviewer via ACP (SEPARATE session — cannot see Builder's reasoning)
sessions_spawn:
  runtime: "acp"
  mode: "run"
  task: |
    [Paste contents of agents/reviewer.md here]
    
    ---
    
    PROJECT PATH: [absolute path]
    
    Read SPRINT.md and BUILDER_REPORT.md, then review all changed files.
    Score each dimension 1-10 and write REVIEWER_REPORT.md.
  cwd: "[project path]"
```

#### Your scoring (always do this, even without Reviewer)

| Dimension | Weight | Score | Notes |
|-----------|--------|-------|-------|
| **Functionality** — works as specified? | 30% | /10 | |
| **Code Quality** — clean, DRY, documented? | 25% | /10 | |
| **Security** — auth, CORS, validation? | 25% | /10 | |
| **Edge Cases** — errors, empty, timeout? | 20% | /10 | |

**Weighted total = F×0.3 + Q×0.25 + S×0.25 + E×0.2**

- **≥ 7.0** → PASS → go to Phase 5
- **5.0 - 6.9** → ITERATE → go to Phase 4
- **< 5.0** → REWRITE → rewrite SPRINT.md with more context, restart Phase 2
- **Security = 1** (auth bypass, secret leak) → automatic FAIL regardless of total

### Phase 4: ITERATE (max 3 rounds)

Write REVIEW.md in the project root:

```markdown
# Review: Round [N] — Score: [X/10]

## Critical Issues (must fix)
1. [FILE:LINE] — [specific issue] — [how to fix]

## Improvements
1. [specific improvement needed]

## What Was Done Well
1. [positive feedback — calibrates the Builder]
```

Spawn Builder again with review context:

```
sessions_spawn:
  runtime: "acp"
  mode: "run"
  task: |
    You are a Builder in round [N] of iteration.
    
    Read SPRINT.md for the original spec.
    Read REVIEW.md for feedback on your previous work.
    Fix all Critical Issues. Improve what's listed.
    Update BUILDER_REPORT.md.
  cwd: "[project path]"
```

Return to Phase 3. **Max 3 rounds.** If still failing after 3, escalate to the user.

### Phase 5: SHIP

1. **Commit:**
```bash
git add -A
git commit -m "[type]: [description]

Sprint: [feature name]
Score: [X.X/10] in [N] round(s)
Builder: Claude Code ACP"
```

2. **Push:** `git push`

3. **Deploy** if applicable (follow project-specific process)

4. **Write HARNESS_REPORT.md:**

```markdown
# Harness Report: [Feature Name]

## Summary
- Sprint type: [backend-api / frontend-component / fullstack]
- Rounds: [N]
- Final score: [X.X/10]
- Total time: [estimate]

## What the Builder Got Right
[What worked on first try]

## What Required Iteration
[What was caught in review and fixed]

## Lessons Learned
[Patterns to carry to next sprint — update SKILL.md anti-patterns if needed]

## Artifacts
- SPRINT.md
- BUILDER_REPORT.md
- REVIEWER_REPORT.md (if used)
- REVIEW.md (if iterated)
```

5. **Notify user** via their channel (Telegram/Discord/etc.)

6. **Archive as example** — copy all artifacts to `examples/[project]-[feature]/`

## Sprint Sizing Guide (Data from Production)

| Sprint Type | Max Lines | Max Files | Success Rate | Notes |
|------------|----------|----------|-------------|-------|
| Backend API endpoint | 100 | 2 | **95%+** | Include schema + patterns |
| Backend refactor | 150 | 3 | **80%+** | Clear before/after spec |
| Frontend component | 100 | 1 | **70%+** | ONE component per sprint |
| Frontend page rewrite | 300+ | 3+ | **<30%** | ⚠️ ALWAYS split |
| Full-stack feature | any | any | **<20%** | ⚠️ ALWAYS split backend + frontend |

**Iron rule:** >2 files OR >150 lines → split into multiple sprints.

## Anti-Patterns (Learned from 8+ Production Sprints)

| ❌ Bad | ✅ Good |
|--------|---------|
| Skip SCOUT, jump to BUILD | Read code thoroughly, write detailed SPRINT.md |
| Vague criteria ("it works") | Specific criteria ("returns 200 with JSON matching schema") |
| Giant sprint (10+ criteria) | 3-5 focused criteria per sprint |
| Frontend + backend in one sprint | Split. Always. |
| Trust Builder blindly | Run mechanical checks + score every time |
| Same session builds and reviews | Separate ACP sessions for review |
| No code examples in SPRINT.md | Paste 1-2 real examples from the codebase |
| Skip DO NOT BREAK list | List every existing feature that must survive |
| Auth imports without checking circular deps | Test full app startup after auth changes |
| Replace data source without fallback | Keep fallback to previous working API |
| Rewrite page without regression list | SPRINT.md must list all existing routes/features |

## Integration with OpenClaw

This harness is designed to run natively on OpenClaw:

| Need | OpenClaw Provides | DeerFlow Equivalent |
|------|------------------|-------------------|
| Builder execution | `sessions_spawn(runtime:"acp")` | LangGraph subagent + Docker |
| Code sandbox | `exec` tool | Docker sandbox provider |
| Memory | `memory_search` / `memory_get` | Custom memory system |
| File access | `read` / `write` / `edit` tools | Sandbox filesystem |
| User notification | `message` tool (Telegram/Discord) | IM channel integrations |
| Background tasks | `sessions_yield` + subagent polling | LangGraph async nodes |

**No Docker. No LangGraph. No Python virtualenv. Just OpenClaw + Claude Code.**

## Quick Reference

```
# Full pipeline in 5 commands:

# 1. Scout
read [project files]
write SPRINT.md

# 2. Build
sessions_spawn → Builder reads SPRINT.md → writes code + BUILDER_REPORT.md

# 3. Review
exec: py_compile / tsc --noEmit / grep
sessions_spawn → Reviewer reads code → writes REVIEWER_REPORT.md
Score 4 dimensions

# 4. Iterate (if < 7.0)
write REVIEW.md
sessions_spawn → Builder reads REVIEW.md → fixes issues

# 5. Ship (if ≥ 7.0)
exec: git commit + push + deploy
write HARNESS_REPORT.md
message: notify user
```
