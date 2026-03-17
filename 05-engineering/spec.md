It looks like file write permissions aren't granted for this path. You can either:

1. **Approve the write** when prompted — the spec is ready to go
2. **Copy the output below** directly into `05-engineering/spec.md`

Here's the full spec:

---

# Technical Spec

## Overview

InsiderEdge is a two-sided pipeline: a background ingestion/scoring system that continuously pulls Form 4 filings from SEC EDGAR, classifies and scores them, and a web API + frontend that surfaces the scored feed and triggers email alerts to subscribers.

Three distinct runtime concerns:

1. **Ingestion workers** — poll EDGAR, parse XML, classify transactions, write to Postgres
2. **Scoring workers** — recalculate `InsiderScore` on new transaction inserts using historical price data
3. **Web server** — FastAPI serving the frontend (Next.js SSR) and API routes; handles auth, billing, and alert configuration

All three run as Docker containers on Fly.io. Workers are scheduled via Fly.io cron machines.

---

## Stack

| Layer | Choice | Rationale |
|---|---|---|
| API / backend | Python 3.12 + FastAPI | Async support for ingestion, matches team background |
| Frontend | Next.js 14 (App Router) + Tailwind CSS | Server components, fast iteration, enforces minimal aesthetic |
| Database | PostgreSQL 16 (Supabase managed) | Relational structure fits scoring queries; Supabase adds auth + RLS |
| Auth | Supabase Auth | Email+password + Google OAuth; no custom auth code |
| Background jobs | Python scripts + Fly.io cron machines | 300–500 filings/day is low throughput; no queue needed in v1 |
| Email | Resend | Simple REST API, good deliverability |
| Billing | Stripe | Subscriptions, webhooks, customer portal |
| Market data | Polygon.io Starter ($29/month) | Historical OHLC for score calculation |
| Analytics | PostHog Cloud | Product events + signal outcome tracking |
| Infrastructure | Docker + Fly.io | Single-region (us-east), simple ops |

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                        Fly.io (us-east)                  │
│                                                          │
│  ┌──────────────┐   ┌──────────────┐  ┌──────────────┐  │
│  │  web server  │   │  ingest      │  │  scorer      │  │
│  │  FastAPI +   │   │  worker      │  │  worker      │  │
│  │  Next.js     │   │  (cron 5min) │  │  (cron 15min)│  │
│  └──────┬───────┘   └──────┬───────┘  └──────┬───────┘  │
│         └──────────────────┼──────────────────┘          │
│                    ┌───────▼────────┐                    │
│                    │  PostgreSQL    │                    │
│                    │  (Supabase)    │                    │
│                    └────────────────┘                    │
└─────────────────────────────────────────────────────────┘
         │                  │                  │
    Supabase Auth      Polygon.io          Resend / Stripe
```

**Request flow:** Next.js App Router → FastAPI `/api/*` → JWT check + tier gate → JSON response → server component render.

**Ingestion flow:** Ingest worker polls EDGAR RSS/submissions API → parses XML → classifies `is_discretionary` → upserts `transactions` → writes insider CIK to `score_queue` → scorer reads queue → fetches Polygon prices → recalculates `insider_scores` → alert worker dispatches Resend emails for matched thresholds.

---

## Data Model

```sql
CREATE TABLE companies (
    cik             INTEGER PRIMARY KEY,
    ticker          VARCHAR(10),
    name            TEXT NOT NULL,
    gics_sector     TEXT,
    gics_industry   TEXT,
    updated_at      TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE insiders (
    cik             INTEGER PRIMARY KEY,
    name            TEXT NOT NULL,
    updated_at      TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE insider_roles (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    insider_cik     INTEGER NOT NULL REFERENCES insiders(cik),
    company_cik     INTEGER NOT NULL REFERENCES companies(cik),
    title           TEXT,
    is_current      BOOLEAN DEFAULT true,
    UNIQUE (insider_cik, company_cik)
);

CREATE TABLE transactions (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    insider_cik         INTEGER NOT NULL REFERENCES insiders(cik),
    company_cik         INTEGER NOT NULL REFERENCES companies(cik),
    accession_number    VARCHAR(25) NOT NULL,
    filed_date          DATE NOT NULL,
    trade_date          DATE NOT NULL,
    transaction_code    CHAR(1) NOT NULL,       -- P=purchase, S=sale
    shares              NUMERIC(18,4) NOT NULL,
    price_per_share     NUMERIC(18,4),
    total_value         NUMERIC(18,2),
    is_discretionary    BOOLEAN NOT NULL DEFAULT false,
    exclusion_reason    TEXT,                   -- '10b5-1', 'option_exercise', 'tax_withholding', null
    raw_xml             TEXT,
    created_at          TIMESTAMPTZ DEFAULT now(),
    UNIQUE (accession_number, trade_date, transaction_code, shares)
);

CREATE INDEX idx_transactions_insider_cik       ON transactions(insider_cik);
CREATE INDEX idx_transactions_filed_date        ON transactions(filed_date DESC);
CREATE INDEX idx_transactions_discretionary     ON transactions(is_discretionary, filed_date DESC);

CREATE TABLE insider_scores (
    insider_cik             INTEGER PRIMARY KEY REFERENCES insiders(cik),
    composite_score         SMALLINT,           -- 0-100, NULL if insufficient history
    company_win_rate_1m     NUMERIC(5,4),
    company_win_rate_3m     NUMERIC(5,4),
    company_win_rate_6m     NUMERIC(5,4),
    company_win_rate_12m    NUMERIC(5,4),
    industry_win_rate_1m    NUMERIC(5,4),
    industry_win_rate_3m    NUMERIC(5,4),
    industry_win_rate_6m    NUMERIC(5,4),
    industry_win_rate_12m   NUMERIC(5,4),
    sample_size             INTEGER NOT NULL DEFAULT 0,
    has_sufficient_history  BOOLEAN GENERATED ALWAYS AS (sample_size >= 5) STORED,
    last_calculated         TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE coupled_trades (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    insider_cik         INTEGER NOT NULL REFERENCES insiders(cik),
    sell_transaction_id UUID NOT NULL REFERENCES transactions(id),
    buy_transaction_id  UUID NOT NULL REFERENCES transactions(id),
    window_days         INTEGER NOT NULL,
    flagged_at          TIMESTAMPTZ DEFAULT now(),
    UNIQUE (sell_transaction_id, buy_transaction_id)
);

CREATE TABLE users (
    id                    UUID PRIMARY KEY REFERENCES auth.users(id),
    email                 TEXT NOT NULL UNIQUE,
    tier                  TEXT NOT NULL DEFAULT 'free' CHECK (tier IN ('free', 'paid')),
    score_alert_threshold SMALLINT NOT NULL DEFAULT 70 CHECK (score_alert_threshold BETWEEN 0 AND 100),
    stripe_customer_id    TEXT,
    stripe_sub_id         TEXT,
    created_at            TIMESTAMPTZ DEFAULT now(),
    updated_at            TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE alerts (
    id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id           UUID NOT NULL REFERENCES users(id),
    transaction_id    UUID NOT NULL REFERENCES transactions(id),
    sent_at           TIMESTAMPTZ DEFAULT now(),
    resend_message_id TEXT,
    UNIQUE (user_id, transaction_id)
);

CREATE TABLE signal_outcomes (
    id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    transaction_id   UUID NOT NULL REFERENCES transactions(id) UNIQUE,
    return_30d       NUMERIC(8,4),
    spy_return_30d   NUMERIC(8,4),
    outperformed_spy BOOLEAN,
    recorded_at      TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE score_queue (
    insider_cik INTEGER PRIMARY KEY REFERENCES insiders(cik),
    queued_at   TIMESTAMPTZ DEFAULT now()
);
```

---

## API Design

Base path: `/api/v1`. All responses `application/json`. Auth via `Authorization: Bearer <supabase_jwt>`. Free tier: `filed_date <= now() - interval '48 hours'` applied server-side; response envelope includes `"data_delay_hours": 48`.

**`GET /api/v1/feed`**
Query: `limit` (default 25, max 100), `cursor` (keyset on `filed_date DESC, id`), `min_score`
```json
{
  "data_delay_hours": 0,
  "next_cursor": "2026-03-14T18:22:00Z_uuid",
  "items": [{
    "transaction_id": "uuid",
    "filed_date": "2026-03-14",
    "trade_date": "2026-03-13",
    "insider": { "cik": 1234567, "name": "Jane Smith", "title": "CEO" },
    "company": { "cik": 789012, "ticker": "ACME", "name": "Acme Corp", "gics_sector": "Technology" },
    "shares": 50000, "price_per_share": "24.50", "total_value": "1225000.00",
    "score": { "composite": 82, "has_sufficient_history": true },
    "signals": ["company_impact", "coupled_trade"]
  }]
}
```

**`GET /api/v1/insiders/{cik}`**
```json
{
  "cik": 1234567, "name": "Jane Smith",
  "roles": [{ "company_ticker": "ACME", "title": "CEO", "is_current": true }],
  "score": {
    "composite": 82, "has_sufficient_history": true, "sample_size": 14,
    "company_win_rates": { "1m": 0.71, "3m": 0.79, "6m": 0.71, "12m": 0.64 },
    "industry_win_rates": { "1m": 0.64, "3m": 0.71, "6m": 0.64, "12m": 0.57 }
  },
  "recent_transactions": [{
    "transaction_id": "uuid", "trade_date": "2026-03-13", "ticker": "ACME",
    "transaction_code": "P", "shares": 50000, "total_value": "1225000.00",
    "coupled_trade": { "sell_ticker": "OTHCO", "sell_date": "2026-02-28", "window_days": 13 }
  }]
}
```

Free tier: `recent_transactions` limited to last 90 days + 48hr delay. Paid: full history.

**`GET /api/v1/alerts/settings`** — returns `{ score_alert_threshold: 75 }`
**`POST /api/v1/alerts/settings`** — body `{ score_alert_threshold: 75 }`, paid tier only

**`POST /api/v1/billing/checkout`** — returns `{ checkout_url: "https://checkout.stripe.com/..." }`
**`POST /api/v1/billing/portal`** — returns `{ portal_url: "..." }`
**`POST /api/v1/webhooks/stripe`** — handles `customer.subscription.created/deleted`, `invoice.payment_failed`; updates `users.tier`

Auth routes (sign-up, sign-in, password reset) handled by Supabase Auth client SDK in Next.js — no custom routes needed.

---

## Infrastructure

### Dockerfiles

**`Dockerfile.web`** — FastAPI + Uvicorn + Next.js static build
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN cd frontend && npm ci && npm run build
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080"]
```

**`Dockerfile.worker`** — single image, entrypoint set per Fly.io cron machine
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
# Ingest machine: python -m workers.ingest   (schedule: */5 * * * *)
# Scorer machine: python -m workers.scorer   (schedule: */15 * * * *)
```

### Fly.io

- `web` app: 512MB shared-CPU-1x, single instance in `iad`
- `ingest` cron machine: boots every 5 min, runs, exits
- `scorer` cron machine: boots every 15 min, runs scorer then alert dispatcher, exits
- Health check: `GET /api/v1/health` → `{"status": "ok"}`

Secrets managed via `fly secrets set`. Migrations via one-off machine running `alembic upgrade head` pre-deploy.

### Monitoring

- Fly.io health checks on `/api/v1/health`
- `worker_heartbeats` table; staleness > 30 minutes triggers email alert
- PostHog for ingestion error rate tracking

---

## Open Questions

1. **Frontend topology**: Co-locate Next.js in the `web` container (v1 recommendation) or separate Fly.io app? Split only if scaling diverges.

2. **Historical backfill scoring**: Initial backfill will queue thousands of insiders simultaneously. Need a bulk-score path that bypasses `score_queue` and processes in batches before launch.

3. **Polygon rate limits**: Starter plan is 5 calls/minute on some endpoints. Fine for ongoing ingestion (300–500/day), but the backfill needs a token-bucket rate limiter and the bulk OHLC endpoint (`/v2/aggs/ticker/{ticker}/range/...`).

4. **EDGAR polling**: Use submissions JSON API for delta polling keyed on `accession_number`. RSS as fallback. Respect 10 req/sec limit with a short sleep between batches.

5. **Coupled trade false positives**: In v1, surface the flag with window duration explicit ("Sold OTHCO 13 days before this buy") rather than filtering programmatically. Let users judge context.

6. **Supabase connection limits**: Free tier may bottleneck with multiple workers holding idle connections. If hit, add PgBouncer or migrate to Railway/Fly Postgres.
