# Sprint Template: Full-Stack Feature

> Use for: Features that need both backend API + frontend UI
> ⚠️ ALWAYS split into 2+ sprints — never build full-stack in one sprint
> Expected success rate for single sprint: <20% (this is why we split)

## Feature: [Feature Name]

## Splitting Strategy

### Sprint A: Backend API (run first)
Use `templates/backend-api.md` for this sprint.
Goal: API endpoint working, tested with curl, returns correct data.

### Sprint B: Frontend UI (run after Sprint A passes)
Use `templates/frontend-component.md` for this sprint.
Goal: Component fetches from Sprint A's API and renders correctly.

### Why Split?
1. **Builder context** — Backend and frontend have different patterns, conventions, and testing
2. **Verification** — We can curl the API before building UI on top of it
3. **Iteration** — If the API design is wrong, we fix it before frontend depends on it
4. **Success rate** — Backend-only sprints succeed 95%+, frontend-only 70%+. Combined <20%.

## Feature Spec

### Goal
[What user-facing feature are we building?]

### User Flow
1. User does X
2. Frontend calls API Y
3. Backend processes and returns Z
4. Frontend displays Z

### API Contract
```
POST /api/feature
Request: { "field": "value" }
Response: { "data": { "result": "..." } }
```

### UI Design
[Description or screenshot reference]

## Sprint A: Backend
[Fill in using backend-api.md template]

## Sprint B: Frontend
[Fill in using frontend-component.md template]
[Add API endpoint from Sprint A to the Context section]

## DO NOT BREAK (applies to both sprints)
- [ ] All existing API endpoints
- [ ] All existing pages
- [ ] Auth flow
- [ ] Mobile responsiveness
