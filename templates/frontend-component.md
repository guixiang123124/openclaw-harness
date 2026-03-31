# Sprint Template: Frontend Component

> Use for: Adding/modifying React/Next.js components
> Ideal size: <100 lines per component, ONE component per sprint
> Expected success rate: 70%+ (frontend is harder for Builders)

## Sprint: [Component Name]

## Goal
[One sentence — what component are we building?]

## Success Criteria
- [ ] Component renders without errors
- [ ] Component handles loading state
- [ ] Component handles error state
- [ ] Component handles empty data state
- [ ] Mobile responsive (if applicable)
- [ ] tsc --noEmit passes
- [ ] No console.log / TODO / any types in code

## File Scope
**Can modify:**
- `[path/to/components/NewComponent.tsx]` (create new)
- `[path/to/page.tsx]` (to import the component)

**Must NOT touch:**
- Layout files (layout.tsx)
- Auth components
- Global styles
- Other existing components

## Context
[Paste relevant info here:]
- What framework? (Next.js App Router / Pages Router / React)
- What styling? (Tailwind / CSS Modules / styled-components)
- What state management? (useState / zustand / context)
- What API client? (fetch / axios / SWR)

### Design Reference
[Describe the visual design or paste a screenshot URL]

### Existing Component Pattern (MUST follow)
```tsx
// Paste 1 existing component that's similar in structure
```

### API Endpoint (if fetching data)
```
GET /api/endpoint
Response: { "data": [...], "total": 42 }
```

## Technical Notes
- Color scheme: [e.g., dark mode, CSS variables]
- Responsive breakpoints: [e.g., md:768px]
- [Any other conventions]

## DO NOT BREAK
- [ ] All existing pages still render
- [ ] Navigation/routing unchanged
- [ ] Auth flow (login/logout) unchanged
- [ ] No layout shift on existing pages
- [ ] Mobile responsiveness of existing components

## ⚠️ Frontend Sprint Rules
1. ONE component per sprint — do not rewrite an entire page
2. If >300 lines needed, split into multiple sprints
3. Builder must verify render by reading the component tree, not just compiling
4. Prefer composition over inheritance — small, focused components
