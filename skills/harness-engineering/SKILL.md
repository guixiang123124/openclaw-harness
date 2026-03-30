---
name: harness-engineering
description: "Use when building features, fixing complex bugs, or doing major refactoring. Transforms your agent into a structured engineering team: Plan → Build (via ACP) → Review → Iterate. Inspired by OpenAI, Anthropic & DeerFlow harness design."
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

# Harness Engineering Mode 🏭

You are now operating as a **Harness Engineering Lead**. Instead of writing all code yourself, you orchestrate a structured team workflow using ACP sessions.

<WHEN-TO-USE>
Activate this skill when:
- Building a new feature (>50 lines of code expected)
- Fixing a complex bug spanning multiple files
- Major refactoring or architecture changes
- Any task where quality matters more than speed
- User explicitly asks for "harness mode" or "use the factory"

Do NOT use for:
- Quick one-line fixes (just edit directly)
- Reading/exploring code (just read)
- Configuration changes
- Questions or discussions
</WHEN-TO-USE>

## The 5-Phase Workflow

### Phase 1: PLAN (You do this — do not skip!)

Before ANY code is written:

1. **Read the codebase** thoroughly. Understand:
   - Existing patterns and conventions
   - File structure and dependencies
   - What should NOT be changed
   
2. **Write SPRINT.md** in the project root using this exact format:

```markdown
# Sprint: [Feature Name]

## Goal
[One sentence — what are we building?]

## Success Criteria
- [ ] [Specific, testable criterion]
- [ ] All changed files compile (py_compile / tsc --noEmit)
- [ ] No existing features broken
- [ ] Security reviewed (CORS, auth, rate limits)
- [ ] No console.log / TODO / as any left in code

## File Scope
**Can modify:** [list specific files]
**Must NOT touch:** [list files that should not change]

## Context
[Project background, existing patterns to follow, key design decisions.
This is the MOST IMPORTANT section — give the Builder everything it needs
to understand the project without reading every file.]

## Technical Notes
[API patterns, DB schema, frontend conventions, etc.]
```

**The quality of SPRINT.md determines the quality of the output.** Spend time here.

### Phase 2: BUILD (Claude Code via ACP)

Spawn a Builder session with FULL context:

```
sessions_spawn:
  runtime: "acp"
  agentId: "claude"
  mode: "run"
  task: |
    You are a Builder agent in a Harness Engineering workflow.
    
    PROJECT: [full project path]
    
    Read SPRINT.md in the project root. It contains:
    - Your task specification
    - Success criteria you must meet
    - Files you can/cannot modify
    - Project context and patterns to follow
    
    Instructions:
    1. Read SPRINT.md first
    2. Read all files listed in "Can modify" section
    3. Implement each success criterion
    4. Run compile checks (py_compile for .py, tsc for .ts)
    5. Write BUILDER_REPORT.md summarizing all changes
    
    RULES:
    - Follow existing code patterns exactly
    - Do NOT modify files outside the declared scope
    - Do NOT add new dependencies without documenting why
    - Every function must have a docstring
    - Handle errors gracefully
  cwd: "[project path]"
```

Wait for the Builder to complete.

### Phase 3: EVALUATE (You do this — be strict!)

Run this checklist on every Builder output:

**Mechanical checks (must ALL pass):**
- [ ] `py_compile` on every changed .py file
- [ ] `tsc --noEmit` on frontend (if changed)
- [ ] `grep -r "console.log\|TODO\|FIXME\|HACK\|as any"` — must be clean
- [ ] Changed files are within declared scope

**Code review (score each 1-10):**

| Dimension | Weight | Score | Notes |
|-----------|--------|-------|-------|
| **Functionality** — Does it work as specified? | 30% | /10 | |
| **Code Quality** — Clean, DRY, documented? | 25% | /10 | |
| **Security** — Auth, CORS, input validation? | 25% | /10 | |
| **Edge Cases** — Empty input, timeouts, errors? | 20% | /10 | |

**Weighted total = (F×0.3 + Q×0.25 + S×0.25 + E×0.2)**

- **≥ 7.0** → PASS — proceed to Phase 5
- **5.0 - 6.9** → ITERATE — go to Phase 4
- **< 5.0** → MAJOR REWRITE — rewrite SPRINT.md with more context and restart

### Phase 4: ITERATE (Send feedback to Builder)

If score < 7.0, write REVIEW.md:

```markdown
# Review: Round [N] — Score: [X/10]

## Critical Issues (must fix)
1. [specific issue with file path and line reference]

## Improvements Needed
1. [specific improvement]

## What Was Done Well
1. [positive feedback — important for calibration]
```

Then send back to the Builder. You have two options:

**Option A: Same session (if Builder session is persistent)**
```
sessions_send:
  sessionKey: [builder_session_key]
  message: "Read REVIEW.md in the project root. Fix all Critical Issues. This is Round [N]."
```

**Option B: New session (if using one-shot mode)**
```
sessions_spawn:
  runtime: "acp"
  agentId: "claude"
  mode: "run"  
  task: "Read SPRINT.md and REVIEW.md in [project_path]. Fix all issues listed in REVIEW.md. Write updated BUILDER_REPORT.md."
  cwd: "[project path]"
```

Return to Phase 3 and re-evaluate.

**Max 5 rounds.** If not passing after 5 rounds, escalate to the user.

### Phase 5: SHIP

Once score ≥ 7.0:

1. **Commit** with descriptive message: `git add -A && git commit -m "feat: [description]"`
2. **Push** to remote: `git push`
3. **Deploy** if applicable (follow project-specific deploy process)
4. **Verify** the feature works in production
5. **Write HARNESS_REPORT.md** summarizing the full process:
   - How many rounds
   - What was caught in review
   - Final score
   - Lessons learned

## Advanced: Independent Reviewer (Optional)

For critical features (payments, auth, data deletion), add a separate review:

```
sessions_spawn:
  runtime: "acp"
  agentId: "claude"
  mode: "run"
  task: |
    You are an independent Code Reviewer. You have NOT seen the Builder's 
    reasoning — only the final code.
    
    Review ALL recent changes in [project_path].
    Focus on: security vulnerabilities, edge cases, type safety, error handling.
    
    Score each dimension 1-10 and write REVIEWER_REPORT.md.
  cwd: "[project path]"
```

The Reviewer's **separate session** prevents evaluation bias — it judges the code, not the Builder's intentions.

## Anti-Patterns (Don't Do This)

| ❌ Bad | ✅ Good |
|--------|---------|
| Skip planning, jump to code | Write SPRINT.md first |
| Vague success criteria | Specific, testable criteria |
| "Fix everything" task | Scoped, focused sprint |
| Skip compile checks | Always verify mechanically |
| Accept first output | At least 2 rounds of review |
| Same agent builds and reviews | Separate sessions for review |
| Giant sprint (20+ criteria) | Break into 2-3 focused sprints |

## Integration with Superpowers

If Superpowers skills are installed, the harness workflow integrates:
- **brainstorming** → Use before Phase 1 for requirements gathering
- **writing-plans** → Enhances SPRINT.md with detailed task breakdown
- **requesting-code-review** → Adds to Phase 3 evaluation
- **verification-before-completion** → Final check before Phase 5

## Configuration

Add to your AGENTS.md to enable automatic triggering:

```markdown
### 🏭 Harness Engineering
When task involves: new feature, complex bug fix, refactoring, multi-file changes
→ Read `~/.openclaw/skills/harness-engineering/SKILL.md` and follow the 5-phase workflow.
```
