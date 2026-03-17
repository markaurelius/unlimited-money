# Design Brief

> ~1 page. Describe the experience, not the aesthetics. Visuals live in Figma; decisions live here.

## Design Principles

1. **Signal over noise** — every element on screen should earn its place. If it doesn't help the user evaluate a trade, it doesn't belong in v1.
2. **Trust through clarity** — financial tools live or die on credibility. Labels, scores, and data provenance should be explicit, not implied. Never make a number look more confident than it is.
3. **Readable at a glance** — the feed is the product. A user should be able to scan 10 insider transactions in 30 seconds and know which ones deserve attention.

## Key Flows

### Flow 1: New user evaluates the product (free tier)
1. Lands on home — sees the feed with recent high-score insider transactions (48hr delayed, clearly labeled)
2. Sees each card: insider name + title, company, score badge, transaction summary, which signal fired (company impact / industry impact / coupled trade)
3. Clicks an insider card → insider profile page: score breakdown at 1/3/6/12mo, historical transactions, win rate visualization
4. Hits the upgrade prompt — "Get real-time alerts when score > X"
5. Signs up, enters payment, sets alert threshold

### Flow 2: Returning paid user receives an alert
1. Gets email: "[Insider Name] (Score: 82) bought $450k of TICKER — Company Impact: 71% positive at 6mo"
2. Clicks through to insider profile page
3. Sees full score breakdown and transaction history to evaluate context
4. No further action required — the alert is the product

### Flow 3: Returning paid user checks the feed
1. Logs in → feed, sorted by score descending, last 7 days
2. Scans cards, clicks into any that are interesting
3. May adjust alert threshold in settings

## Component Inventory

- [ ] Feed card — insider name/title, company, score badge, transaction summary, signal type label, trade date
- [ ] Insider profile page — score breakdown (4 dimensions × 4 time horizons), transaction history table, coupled trade flags
- [ ] Score badge — simple numeric score (0–100) with color band (green/yellow/red), tooltip explaining methodology
- [ ] Alert settings — score threshold slider, email confirmation
- [ ] Upgrade prompt / paywall — surfaces on real-time data, minimal friction
- [ ] Auth screens — sign up, log in, forgot password (standard, no custom design needed)

## Design References

- Levels.fyi — clean data presentation, good use of whitespace, no decorative elements
- Linear.app — minimal nav, content-first, fast feel
- Aesthetic: light mode default, optional dark mode later, monospace or semi-monospace for numbers

## Responsive / Platform Notes

- Desktop-first for v1. The feed and insider profile are information-dense enough that mobile is a second-class experience.
- Mobile: feed cards should be readable, but full score breakdown can truncate on small screens.
- No native app in v1 or v2. Email is the mobile surface.

---

## Open Questions

- Should the score be a single composite number (0–100) or displayed as a set of components? A composite is easier to act on; components are more transparent. Recommendation: show composite as the headline, components on the profile page.

## Decisions Made

- Web-only, desktop-first for v1
- Minimal aesthetic — no decorative elements, data and whitespace only
- Email is the primary mobile surface (alerts), not a responsive web app
- Score badge is the primary visual element — needs to be instantly scannable

---

## Stack Recommendation

Given the product shape (data pipeline + simple web UI + email alerts), your background (data/backend engineering, Python), and the build-first constraint:

- **Backend:** Python + FastAPI — matches your background, fast to build, good async support for the ingestion pipeline
- **Frontend:** Next.js + Tailwind CSS — server components keep it simple, Tailwind enforces the minimal aesthetic, fast to iterate
- **Database:** PostgreSQL (managed via Supabase or Railway) — structured data, good for the time-series score calculations
- **Background jobs:** Celery + Redis, or simple cron workers on Fly.io — Form 4 ingestion and score recalculation don't need anything exotic
- **Hosting:** Fly.io — simple, cheap, good for a solo operator running background workers alongside a web server
- **Auth:** Supabase Auth or Auth.js — no reason to build this
- **Email:** Resend — simple API, good developer experience
- **Billing:** Stripe — standard
