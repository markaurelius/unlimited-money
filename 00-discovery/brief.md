# Discovery Brief

> ~1 page. Answer concretely. If you can't answer a question, that's important signal.

## The Problem

Retail investors who actively track corporate insider trades can't get fast, accurate, comprehensive signals from existing tools. Unusual Whales — the most popular option — is primarily built around congressional trade tracking, treats corporate Form 4 data as secondary, introduces delays between filing and alert, and doesn't clearly document data provenance or coverage. Free tools like OpenInsider are comprehensive but basic: no filtering, no noise removal, no alerting. The result is a signal that's either stale, noisy (full of automatic 10b5-1 plan sales and option exercises that aren't meaningful), or buried inside a product optimized for something else.

## The Users

Primary user: a retail investor with a self-directed taxable brokerage account ($50k–$500k), who already follows financial communities (Reddit, Twitter/X), already pays for tools like Unusual Whales, and is actively looking for edges. They're not a quant — they won't build their own pipeline — but they're sophisticated enough to know that raw insider data needs filtering to be useful. The founder is this user.

## Why Now

- The SEC's EDGAR system now exposes a public REST API (data.sec.gov) that makes Form 4 ingestion tractable without expensive data licensing
- Compute costs make real-time pipeline processing viable for a solo operator
- Unusual Whales' rapid rise and congressional focus has left a gap specifically in corporate insider coverage
- The academic signal is well-established (multiple papers show discretionary insider buys predict outperformance) but no retail-accessible, signal-filtered product has been built on top of it

## What Success Looks Like

**First milestone (pre-customer):** A backtested dataset demonstrating that the filtered signal — clean discretionary insider buys, stripped of 10b5-1 plan sales, option exercises, and tax withholding transactions — outperforms a benchmark strategy (e.g., SPY buy-and-hold) over a meaningful historical window. This proves the product premise before any marketing spend.

**At 12 months:** A live alerting product with paying subscribers, where the value proposition is validated signal quality, not just data access. Exact subscriber count is secondary to proving the signal works.

---

## Open Questions

- What lookback period and methodology makes a statistically credible backtest? (Need enough trades to be significant, not cherry-picked)
- How reliably can 10b5-1 plan sales be identified and filtered from raw Form 4 data? (They're disclosed but inconsistently formatted)
- What's the precise latency of EDGAR's public API after a Form 4 is submitted — minutes, hours?
- Is there a meaningful difference in signal quality between C-suite buys vs. director buys vs. 10%+ shareholders?

## Decisions Made

- Primary user: retail investor, not developer/quant
- First milestone is signal validation via backtest, not customer acquisition
- Product differentiator: speed + noise filtering (remove non-discretionary transactions) vs. Unusual Whales
