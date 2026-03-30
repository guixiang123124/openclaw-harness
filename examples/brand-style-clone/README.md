# Case Study: Brand Style Clone (FashionFlow)

## What We Built
A feature that scrapes brand product pages (Zara, H&M, Shopify stores), extracts model photos, and lets users replicate each styling with their own product. 

## The 5 Rounds

### Round 1: B+ — Basic Implementation
- 5-strategy scraper (Shopify JSON, JSON-LD, og:image, Jina, HTML mining)
- Image filtering + deduplication
- ✅ Worked but had dependency issues

### Round 2: A- — Dependency Cleanup
- **Found:** Used BeautifulSoup (not in project) and requests (project uses httpx)
- **Fixed:** Replaced with regex parsing + httpx
- **Found:** Jina timeout too long (25s)
- **Fixed:** Reduced to 15s
- Added per-strategy timing logs

### Round 3: A — UX Polish  
- Better error messages with suggested actions
- Source strategy badges ("Found via Shopify API")
- Image placeholder for missing images

### Round 4: A — CRITICAL BUG FOUND 🔴
- **CORS Bug:** `fetch(brandUrl)` from browser blocked by CORS on ~70% of brand CDNs
- Feature was essentially **broken for most real users**
- **Fix:** Added backend image proxy `/brand-clone/fetch-image`
- Also: parallelized 5 strategies (5-8s faster)

### Round 5: A+ — Ship Ready
- SSRF protection (full RFC-1918 coverage)
- Rate limit on proxy endpoint  
- Auth token expiry handling (abort batch + clear message)
- Removed all `as any` casts
- Full integration test simulation (3-succeed-2-fail scenario)

## Key Insight
**Without Round 4, this feature would have shipped broken.** The CORS bug was invisible in code review — only found by thinking about the actual browser execution environment.

## Metrics
- Total code: ~800 lines (service + API + frontend)
- Bugs caught: 2 critical (CORS, SSRF), 4 medium
- Time: ~3 hours across 5 rounds
