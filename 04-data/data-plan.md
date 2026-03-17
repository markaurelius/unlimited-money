# Data Plan

> ~1 page. Decide what you'll measure before you build, not after.
> Instrumentation that isn't planned here usually doesn't get built.

## North Star Metric

**Paid subscriber retention rate (monthly)** — the % of paid subscribers still active 30 days after subscribing. Target: >90% monthly retention (<10% churn). This metric captures both signal quality (users making better-informed trades) and product value simultaneously. If users are making money with the signal, they renew. If they aren't, they cancel.

Secondary validation: **monthly signal accuracy rate** — % of high-score alerts where the underlying stock outperformed SPY at 30 days. This is internally measurable (no user data required) and serves as both a quality check and a retention feature reported back to users.

## Supporting Metrics

| Metric | Definition | Target (90 days) |
|---|---|---|
| Alert click-through rate | % of alert emails where user clicks through to insider profile | > 35% |
| Free-to-paid conversion rate | % of free signups who upgrade within 30 days | > 8% |
| Signal accuracy rate | % of high-score alerts where stock outperformed SPY at 30 days | > 60% (vs. ~50% random baseline) |
| Alerts sent per paid user/week | Average alerts triggered per paid user — proxy for engagement | 2–5 (too few = boring, too many = noisy) |
| Monthly churn rate | % of paid subscribers who cancel in a given month | < 10% |

## Instrumentation Plan

| Event | Trigger | Properties |
|---|---|---|
| `user_signed_up` | On account creation | `source`, `plan` (free/paid) |
| `user_upgraded` | Free → paid conversion | `days_since_signup`, `alert_threshold_set` |
| `alert_sent` | Email alert dispatched | `insider_cik`, `company_ticker`, `insider_score`, `signal_type` |
| `alert_clicked` | User clicks alert email CTA | `alert_id`, `time_since_sent` |
| `insider_profile_viewed` | User views insider profile page | `insider_cik`, `source` (feed/alert/direct) |
| `user_churned` | Subscription cancelled or payment failed | `days_active`, `alerts_clicked_total` |
| `signal_outcome_recorded` | Automated: 30-day return calculated for each alerted transaction | `alert_id`, `return_30d`, `outperformed_spy` (boolean) |

**Analytics tool:** PostHog (self-hosted or cloud) — captures both product analytics and the `signal_outcome_recorded` event without needing a separate data warehouse in v1. Postgres is sufficient for everything else.

## Experiment Design

**First experiment — alert threshold default**

**Question:** Does the default alert score threshold (the score above which a user gets notified) affect retention?
**Hypothesis:** Users set to a higher default threshold (e.g., 75 vs. 60) receive fewer but higher-quality alerts, leading to higher click-through rates and better 30-day retention.
**Metric:** 30-day retention rate and alert click-through rate by threshold cohort.
**Minimum sample size:** ~200 paid users per cohort — hold this experiment until sufficient scale.

Before any A/B testing: the first and most important experiment is the backtest itself. Run the historical signal accuracy analysis before launch. Publish the result (with methodology) on the site. This is the primary conversion tool, not a feature.

## Data Infrastructure

- **Storage:** PostgreSQL — all transactions, scores, users, alerts, and signal outcomes live here. No separate data warehouse needed in v1.
- **Score calculation:** Python scripts run as background jobs; results written back to Postgres. Historical backtest runs once at launch, then incrementally updated.
- **Signal outcome tracking:** A daily cron job pulls 30-day-forward prices for any alerted transaction that is now 30+ days old, records the outcome. No third-party service needed.
- **Querying:** Direct Postgres queries + PostHog for product analytics. No dbt, no Redshift in v1.
- **Alerting (internal ops):** Simple Fly.io health checks + email on ingestion job failure.

---

## Open Questions

- At what signal accuracy rate does the product lose credibility? (Hypothesis: below 55% sustained, users notice and churn; need to define a kill threshold.)
- Should the monthly accuracy report go to all users (free + paid) as a retention/acquisition tool, or paid only?

## Decisions Made

- North star: paid subscriber retention rate
- Signal accuracy rate is both an internal quality metric and a user-facing retention feature (monthly report)
- PostHog for product analytics, Postgres for everything else — no data warehouse in v1
- First real experiment is the pre-launch backtest, not an A/B test
