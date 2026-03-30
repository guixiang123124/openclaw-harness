# Case Study: TrendMuse Full Upgrade

## What We Did
Comprehensive audit and upgrade of TrendMuse, a fashion trend discovery app with 53 API endpoints, from "mostly broken" to production-ready.

## The 5 Rounds

### Round 1: Full Audit
- Tested all 53 API endpoints against live backend
- **Root cause found:** `products` table had 0 rows while `trend_signals` had 1,167
- Two separate storage systems that were never connected
- 25 endpoints returned empty data because of this

### Round 2: Core Data Pipeline
- Built migration script (`trend_signals` → `products` table)
- Scanner now persists to DB (was in-memory, lost on restart)
- Fixed Exa PATH for Linux (was macOS-only `/opt/homebrew/bin`)
- Generator designs + converter sketches now persist to SQLite

### Round 3: CRITICAL BUG FOUND 🔴
- **Frontend URL Bug:** `fetch("/api/generator/redesign")` hit Next.js server, not FastAPI backend
- "Redesign" button silently 404'd in production — users clicked, nothing happened
- **Fix:** Changed to `apiUrl("/api/generator/redesign")`
- Also: enum crash prevention, schema compatibility verification

### Round 4: Production Hardening
- CORS restricted from `*` to specific domains
- Admin auth on migration endpoints
- Trend scoring algorithm (price + recency + image presence)
- Image placeholder for missing images
- All 7 .py files pass py_compile

### Round 5: UI Polish
- Removed all Chinese text from English app (2 files, 12 strings)
- Improved empty state messaging with CTA button
- Product card consistency across pages

## Key Insight
**The most valuable finding was in Round 1** — discovering that two storage systems were never connected. Everything downstream was a consequence of this architectural gap.

## Metrics
- Endpoints audited: 53
- Endpoints fixed: 16 (from empty/broken to functional)  
- Bugs caught: 1 critical (URL routing), 1 architectural (disconnected storage)
- Time: ~2.5 hours across 5 rounds
