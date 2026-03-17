# Product Requirements

> ~2 pages. Be concrete enough that an engineer can estimate scope.
> Every requirement should trace back to a user problem in 00-discovery/brief.md.

## Problem Statement

Retail investors who track corporate insider trades can't get a reliable, filtered signal from existing tools. Unusual Whales is delayed and congress-focused. OpenInsider is raw data with no analysis. No tool scores individual insiders by historical accuracy or surfaces the most analytically interesting signal: when an insider sells one stock and buys another, suggesting capital rotation based on information asymmetry. The result is a noisy, uncontextualized data stream that requires significant manual work to extract actionable insight.

## Users

Self-directed retail investor with a $50k–$500k taxable brokerage account who already follows financial communities, already pays for tools like Unusual Whales, and wants a faster, cleaner insider signal with an analytical layer that tells them whether a specific insider's moves have historically mattered.

## Requirements

### Must Have (v1)

- [ ] Real-time Form 4 ingestion from SEC EDGAR, filtered to remove non-discretionary transactions (10b5-1 plan sales, option exercises, tax withholding transactions)
- [ ] **Insider score**: per-insider track record across four pre-packaged dimensions (see below), calculated on historical data
- [ ] **Company impact signal**: what % of this insider's open-market buys were up in that company's stock at 1, 3, 6, and 12 months
- [ ] **Industry impact signal**: same forward-return analysis but against the insider's sector/industry index
- [ ] **Coupled trade detection**: flag when an insider sells in one company and buys in another within a configurable window (default: 30 days), excluding known non-discretionary sells
- [ ] Feed of recent notable insider transactions ranked by insider score
- [ ] Email alerts: notify user when a high-score insider (score above user-set threshold) makes a new open-market buy
- [ ] Free tier: 48-hour delayed data, last 90 days of filings
- [ ] Paid tier ($20/month): real-time alerts, full history, all analysis signals, coupled trade detection

### Nice to Have (v2)

- [ ] Sector/industry filtering on the feed (user subscribes to specific sectors)
- [ ] Specific company watchlist alerts
- [ ] Weekly digest email summarizing top-scored insider activity
- [ ] Developer API tier ($99-199/month) exposing scored data programmatically
- [ ] Insider "cold streak" detection — flag insiders whose recent trades have underperformed their historical score

### Out of Scope

- In-browser custom query or analysis builder — users consume pre-packaged outputs only
- Portfolio tracking or brokerage integration
- Congressional trade tracking (explicitly differentiated from Unusual Whales)
- Options flow data

## User Stories

1. As a retail investor, I want to see a ranked feed of recent insider buys filtered by insider score so that I can focus on the signals most likely to be meaningful.
2. As a retail investor, I want to receive an email alert when a high-score insider makes an open-market buy so that I can act on the signal without checking the site daily.
3. As a retail investor, I want to see an insider's historical track record (company + industry impact, win rate at 1/3/6/12 months) so that I can judge whether their current trade is worth paying attention to.
4. As a retail investor, I want to see when an insider sold one stock and bought another within 30 days so that I can identify potential capital rotation signals.
5. As a free user, I want to see delayed insider activity with scores so that I can evaluate the product before upgrading.

---

## Open Questions

- What time window defines a "coupled trade"? 30 days is a reasonable default but needs validation — some insiders may have legitimate non-related transactions in that window.
- How do we handle insiders who have too few historical transactions to produce a statistically meaningful score? (Flag as "insufficient history" rather than hiding them.)
- What's the minimum backtest window to publish a score? (Hypothesis: require at least 5 historical open-market buy transactions.)

## Decisions Made

- v1 is a scored feed + email alerts — no in-browser exploration tool
- Four pre-packaged signal types only: company impact, industry impact, trade outcome rates, coupled trades
- Freemium model: 48hr delay on free, real-time on paid
- Congressional trades explicitly out of scope

---

## Engineering Handoff

**Core entities:**
- `Insider` — SEC CIK, name, title, affiliated companies, score components, score updated timestamp
- `Company` — ticker, CIK, name, GICS sector, GICS industry group
- `Transaction` — insider CIK, company CIK, filed date, trade date, transaction type, shares, price per share, total value, is_discretionary (boolean, derived), raw_form4_xml
- `InsiderScore` — insider CIK, company_win_rate_1m/3m/6m/12m, industry_win_rate_1m/3m/6m/12m, sample_size, last_calculated
- `CoupledTrade` — sell_transaction_id, buy_transaction_id, window_days, flagged_date
- `User` — email, tier (free/paid), score_alert_threshold, created_at
- `Alert` — user_id, transaction_id, sent_at

**Key actions/APIs:**
- EDGAR polling job: ingest new Form 4 filings, parse XML, classify discretionary/non-discretionary, write to Transaction
- Score recalculation job: on new transaction insert, recalculate InsiderScore for that insider using historical price data
- Coupled trade detection job: after each new sell transaction, scan same insider's recent buys within window
- `GET /feed` — ranked list of recent discretionary transactions with insider scores, paginated
- `GET /insider/:cik` — insider profile with score breakdown and transaction history
- `POST /alerts` — user sets alert threshold
- Email delivery: triggered alert emails when new transaction matches user threshold

**Integrations required:**
- SEC EDGAR REST API (data.sec.gov) — Form 4 ingestion, no cost
- Market data API (Polygon.io $29/month or Alpaca free tier) — historical + current prices for score calculation
- Email service (Resend or SendGrid) — alert delivery
- Stripe — subscription billing, free/paid tier management

**Performance/scale constraints:**
- Form 4 filings: ~300-500 new filings per day on average. Not high throughput.
- Score recalculation: can be async, does not need to be real-time — within 15 minutes of new filing is acceptable
- Feed latency: < 500ms p95 is sufficient for a daily-use product

**Auth model:**
- Email + password for v1. Google OAuth as an optional addition.

**Deployment target:**
- Single-region cloud (AWS or Fly.io). Background workers for ingestion and scoring. Simple web server for API. Postgres for storage.
