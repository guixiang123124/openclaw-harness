# Example: Weekly Trend Report API (1 Round Pass)

**Project:** TrendMuse — Fashion Intelligence Platform
**Date:** 2026-03-30
**Harness Version:** v0.6

## The Task
Add a `GET /api/trends/weekly-report` endpoint that generates structured weekly trend analysis from a products database.

## Result
- **Rounds:** 1 (passed first attempt!)
- **Score:** 8.55/10
- **Code:** 133 lines across 2 files
- **Time:** ~4.5 minutes total

## Why 1 Round?
The SPRINT.md was highly detailed:
- Listed all 6 specific JSON fields required in the response
- Specified the exact database schema (column names, types)
- Listed all existing endpoints so Builder could match patterns
- Defined clear file scope (what to modify, what NOT to touch)
- Included technical notes (SQL aggregates, trend_score formula)

## Key Lesson
**The quality of SPRINT.md determines the number of rounds.**

| Sprint Quality | Typical Rounds | Example |
|---------------|---------------|---------|
| Vague ("add a report endpoint") | 4-5 rounds | Brand Style Clone initial |
| Good ("add X with fields A, B, C") | 2-3 rounds | TrendMuse migration |
| **Detailed** (schema, patterns, context) | **1 round** | **This example** |

## Files
- `SPRINT.md` — The sprint contract that made 1-round possible
- `BUILDER_REPORT.md` — Builder's output summary
- `EVALUATION.md` — The 4-dimension scoring

## Scoring Breakdown

| Dimension | Weight | Score | Notes |
|-----------|--------|-------|-------|
| Functionality | 30% | 9/10 | All 6 success criteria met |
| Code Quality | 25% | 9/10 | Perfect pattern match, docstrings, COALESCE nulls |
| Security | 25% | 8/10 | Read-only, no injection risk |
| Edge Cases | 20% | 8/10 | Empty DB handled, NULL protection |
| **Weighted** | | **8.55** | **PASS** |
