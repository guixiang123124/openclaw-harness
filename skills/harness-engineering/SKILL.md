---
name: harness-engineering
description: "Harness Engineering mode: transform your agent into a structured engineering team with Planning, Building, and Reviewing phases. Uses ACP to spawn Claude Code for implementation and independent review. Inspired by OpenAI, Anthropic, and DeerFlow harness design."
---

# Harness Engineering

Use this skill when you need to build features, fix complex bugs, or do major refactoring. It transforms a single task into a structured multi-phase engineering workflow.

## When to Use

- Building a new feature (>50 lines of code)
- Fixing a complex bug with multiple files involved
- Major refactoring or architecture changes
- Any task where quality matters more than speed

## Workflow

### Phase 1: Plan (You do this)

1. **Read the codebase** — Understand existing patterns, dependencies, constraints
2. **Write SPRINT.md** — Use template at `{baseDir}/templates/SPRINT.md`
   - Clear goal (one sentence)
   - Success criteria (checkboxes)
   - File scope (what to change, what not to touch)
   - Context (project background, key decisions)
3. **Save SPRINT.md** in the project root

### Phase 2: Build (Claude Code ACP does this)

Spawn a Builder session:
```
sessions_spawn:
  runtime: "acp"
  agentId: "claude"
  mode: "session"
  task: "Read SPRINT.md in [project_path]. Implement all tasks listed. Follow existing code patterns. Write BUILDER_REPORT.md when done."
  cwd: "[project_path]"
```

Wait for completion. The Builder will:
- Read SPRINT.md for context
- Implement each task
- Run compile checks
- Write a report

### Phase 3: Evaluate (You do this)

Run the evaluation checklist:

1. **Compile check**: `py_compile` / `tsc --noEmit` on all changed files
2. **Code review**: Read every changed file
3. **Security scan**: Check CORS, auth, rate limits, SSRF
4. **Edge cases**: What happens on empty input? Timeout? Concurrent access?
5. **Integration**: Does the full user flow work end-to-end?

Score each dimension (see `{baseDir}/references/evaluation-dimensions.md`):
- Functionality (30%)
- Code Quality (25%)
- Security (25%)
- UX / Edge Cases (20%)

**If total score < 7/10**: Write REVIEW.md with specific feedback, send back to Builder:
```
sessions_send:
  sessionKey: [builder_session]
  message: "Read REVIEW.md. Fix all issues listed. This is Round [N]."
```

**If total score ≥ 7/10**: Proceed to deploy.

### Phase 4: Optional Independent Review

For critical features, spawn a separate Reviewer:
```
sessions_spawn:
  runtime: "acp"
  agentId: "claude"
  mode: "run"
  task: "You are a senior code reviewer. Review ALL changes in [project_path] since [commit/time]. Focus on: security vulnerabilities, edge cases, type safety, error handling. Write REVIEWER_REPORT.md."
  cwd: "[project_path]"
```

The Reviewer has a **separate session** — it cannot see the Builder's reasoning, only the code output. This prevents evaluation bias.

### Phase 5: Ship

1. Commit with descriptive message
2. Push to git
3. Deploy to production
4. Verify live functionality

## Red Flags (Stop and Re-evaluate)

- Builder skipped writing tests → Send back
- Changed files outside the declared scope → Send back
- Added new dependencies without justification → Send back
- Score on any single dimension < 5/10 → Major rewrite needed

## Max Rounds

- Default: 5 rounds max
- If not passing after 5 rounds, escalate to human (Xiang)
- Each round should improve the score — if it plateaus, change approach

## Templates

- Sprint contract: `{baseDir}/templates/SPRINT.md`
- Review feedback: `{baseDir}/templates/REVIEW.md`
- Final report: `{baseDir}/templates/REPORT.md`
- Evaluation rubric: `{baseDir}/references/evaluation-dimensions.md`
