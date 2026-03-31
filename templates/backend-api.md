# Sprint Template: Backend API Endpoint

> Use for: Adding new API endpoints, modifying existing ones
> Ideal size: <100 lines changed
> Expected success rate: 95%+

## Sprint: [Endpoint Name]

## Goal
[One sentence — what API endpoint are we building?]

## Success Criteria
- [ ] Endpoint responds with correct status codes (200, 400, 401, 404, 500)
- [ ] Request body validated (required fields, types, ranges)
- [ ] Response matches declared schema
- [ ] Auth required for mutation endpoints (POST/PUT/DELETE)
- [ ] Error responses follow project's error format
- [ ] py_compile passes on all changed files
- [ ] No console.log / TODO / FIXME in code

## File Scope
**Can modify:**
- `[path/to/routes/file.py]`
- `[path/to/models.py]` (if new Pydantic models needed)
- `[path/to/db.py]` (if new DB operations needed)

**Must NOT touch:**
- Auth system (auth.py, security.py)
- Existing endpoint signatures
- Database schema (unless migration is part of the sprint)
- Configuration files

## Context
[Paste relevant info here:]
- What framework? (FastAPI / Express / etc.)
- What's the existing route pattern? (e.g., `/api/v1/resource`)
- How does auth work? (Bearer token, session, API key)
- What's the DB? (SQLite, PostgreSQL, etc.)
- What's the existing error format?

### Existing Patterns (MUST follow)
```python
# Paste 1-2 existing endpoint examples so Builder sees the pattern
```

### DB Schema (relevant tables)
```sql
-- Paste relevant CREATE TABLE statements
```

## Technical Notes
- Response shape: `{"data": ..., "error": null}` or `{"data": null, "error": {"message": "...", "code": "..."}}`
- [Any other conventions]

## DO NOT BREAK
- [ ] All existing endpoints still respond correctly
- [ ] Auth flow unchanged
- [ ] No circular imports introduced
- [ ] Existing tests still pass
