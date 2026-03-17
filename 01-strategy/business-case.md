# Business Case

> ~1-2 pages. Focus on the decisions that affect what you build, not the pitch deck.

## Market

Retail investors who actively track insider trades are a subset of the self-directed investing community. Proxy signals on market size:
- OpenInsider receives millions of visits/month on a free product — large latent audience
- Unusual Whales has 100k+ users paying ~$30/month for a broader product that includes insider data as a secondary feature
- r/investing (3M+ members), r/stocks (4M+), r/thetagang (1.2M) — the communities where this product would spread organically are large

Bottom-up estimate: if 0.5% of the actively engaged self-directed retail investor population ($50k+ portfolio, follows financial communities) would pay $20/month for a focused insider trading signal tool, that's a realistic pool of 50,000–150,000 potential subscribers globally. Year 1 target: 500–2,000 paying users.

$20/month × 1,000 users = $240k ARR. Achievable for a solo operator with strong organic distribution.

## Competition

| Competitor | What they do | Why we're different |
|---|---|---|
| Unusual Whales | Congress + options flow + insider data | Delayed, congress-first, no analysis layer |
| OpenInsider | Comprehensive Form 4 data, free | No filtering, no alerting, no analysis, dated UI |
| InsiderMonkey | Insider data + hedge fund tracking | Content/media site, not a signal tool |
| sec-api.io | Clean EDGAR API for developers | Developer-only, no retail UI, pure data — no insight |

The gap no one fills: **real-time, noise-filtered corporate insider buys with an analysis layer that contextualizes whether the signal is meaningful** — not just data delivery.

## Revenue Model

- **Model:** Freemium subscription
- **Free tier:** Delayed data (48-hour lag), limited to last 30 days of filings, no analysis
- **Paid tier:** $20/month — real-time alerts, full history, analysis layer (signal scoring, cluster buy detection, exec rank weighting)
- **Who pays:** Self-directed retail investors, $50k–$500k portfolio, already paying for tools like Unusual Whales
- **Why they'll pay:** The free tier proves the data exists; the paid tier proves it's actionable. The backtest is the conversion tool — "here's proof this signal has edge" is a compelling upgrade prompt.

No per-usage pricing. Flat subscription keeps acquisition and billing simple for a solo operator.

## OKRs — First 12 Months

**Objective 1: Validate that the signal has real investment edge**
- KR1: Backtested filtered insider buy signal outperforms SPY buy-and-hold over a 5-year window with statistical significance (p < 0.05)
- KR2: Live signal tracked for 90 days post-launch shows consistent directional accuracy vs. benchmark

**Objective 2: Build a sustainable revenue base**
- KR1: 500 paying subscribers by month 12
- KR2: $10k MRR by month 12 (~$120k ARR)

---

## Open Questions

- What free data sources cover historical Form 4 filings back far enough for a statistically credible backtest? (EDGAR goes back to 1993 — coverage is the question, not availability)
- Is the backtest result the primary acquisition hook, or is there a faster "show don't tell" mechanism (e.g., a free weekly newsletter)?
- At what subscriber count does this justify a developer API tier as a second revenue stream?

## Decisions Made

- Revenue model: freemium subscription, $20/month paid tier
- Free tier differentiator: 48-hour delay (not a feature-limited free tier — delay is the lever)
- Conversion mechanism: backtest results as proof of signal quality, surfaced in free tier
