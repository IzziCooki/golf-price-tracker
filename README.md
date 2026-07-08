# Golf Equipment Price Tracker - Implementation Plan

## 1) Product Goal
Build an intelligent dashboard that automatically collects golf equipment prices from multiple sources, normalizes and stores historical data, and presents live insights (price trends, drops, volatility, and alerts) through a modern React UI.

## 2) Success Criteria
- Track prices for at least 3 equipment categories (e.g., drivers, irons, putters).
- Ingest data from at least 3 sources with resilient scraping.
- Maintain historical price snapshots and expose trend analytics.
- Show near real-time updates in UI (WebSocket preferred; polling fallback).
- Support user watchlists, filters, and alert thresholds.
- Ship a deployable MVP with monitoring, logs, and basic tests.

## 3) Architecture Overview
- **Python Scraper Service**
  - Scheduled jobs + source adapters.
  - Rate limiting, retries, backoff, normalization, deduplication.
  - Publishes normalized records to API ingestion endpoint or message queue.
- **Node.js/Express API**
  - REST API for products, price history, analytics, and user preferences.
  - WebSocket channel for live price updates and alert events.
  - Auth (JWT) and role checks.
- **MongoDB**
  - Product catalog, source listings, historical price points, users/watchlists/alerts.
  - Aggregation pipelines for trends and statistics.
- **React + Tailwind Dashboard**
  - Trend charts, top movers, watchlist feed, filter panels.
  - Live updates + optimistic UI interactions.

## 4) Data Model (MongoDB)
- `products`
  - `_id`, `name`, `brand`, `category`, `model`, `specs`, `imageUrl`, `canonicalKey`
- `sources`
  - `_id`, `name`, `baseUrl`, `region`, `currency`, `reliabilityScore`, `active`
- `listings`
  - `_id`, `sourceId`, `productId`, `sourceSku`, `url`, `availability`
- `price_points`
  - `_id`, `listingId`, `productId`, `capturedAt`, `price`, `currency`, `rawPrice`, `shipping`, `inStock`
- `users`
  - `_id`, `email`, `passwordHash`, `settings`
- `watchlists`
  - `_id`, `userId`, `productId`, `targetPrice`, `notifyOnDropPct`, `active`
- `alerts`
  - `_id`, `userId`, `productId`, `triggerType`, `triggerValue`, `sentAt`, `status`

Indexes:
- `price_points`: `(productId, capturedAt desc)`, `(listingId, capturedAt desc)`
- `products`: unique `canonicalKey`
- `watchlists`: `(userId, productId)` unique

## 5) API Contract (Express)
Core endpoints:
- `GET /api/products` (filters: category, brand, q)
- `GET /api/products/:id/history?range=7d|30d|90d`
- `GET /api/analytics/top-movers?window=24h`
- `POST /api/watchlist` / `DELETE /api/watchlist/:id`
- `GET /api/watchlist`
- `POST /api/ingest/price-point` (internal key auth from scraper)

Realtime:
- `ws://.../updates`
  - events: `price_update`, `price_drop_alert`, `source_status`

## 6) Scraper Engine Plan (Python)
### Source Adapter Pattern
- One adapter per website (`BaseAdapter` interface):
  - `fetch_listing_urls()`
  - `parse_listing(html)`
  - `normalize(record)`

### Reliability Requirements
- Request throttling per domain.
- Exponential backoff + retry budget.
- Circuit breaker for repeated failures.
- Structured logging + error buckets by source.
- Idempotent ingestion via deterministic hash (`source + sku + timestamp slice`).

### Scheduling
- APScheduler/Celery beat style cron jobs:
  - High-priority products every 10–15 min.
  - Full catalog every 6–12 hours.

## 7) Frontend Plan (React + Tailwind)
Main pages/components:
- Dashboard overview: KPIs, top price drops, volatility cards.
- Product detail: multi-source historical chart + min/max/avg.
- Watchlist: user-managed tracked products + target thresholds.
- Source health panel: scraper status and freshness.

State/Data:
- React Query for API caching/invalidation.
- WebSocket client for incremental live updates.
- URL-driven filters for sharable views.

Charts:
- Recharts/Chart.js line chart for price trend.
- Bar chart for source comparison.

## 8) Security, Quality, and Ops
- JWT auth, password hashing, CORS policy.
- Input validation with Zod/Joi on API boundaries.
- API rate limiting and ingestion secret for scraper endpoint.
- Centralized logs (API + scraper), request IDs, error reporting.
- Health checks:
  - API: `/health`
  - Scraper heartbeat: last successful run timestamp

## 9) Delivery Phases
### Phase 0 - Setup (1–2 days)
- Monorepo structure (`/api`, `/scraper`, `/web`), Docker Compose for Mongo + services.
- Env management and shared config.

### Phase 1 - Data Pipeline MVP (3–5 days)
- Implement 1 source adapter.
- Ingestion endpoint + Mongo persistence.
- Basic product/history endpoints.

### Phase 2 - Analytics + Realtime (3–4 days)
- Aggregation pipelines for trends/top movers.
- WebSocket push from new price_points.

### Phase 3 - Dashboard UI (4–6 days)
- Overview + product detail + watchlist.
- Filters, charts, live feed.

### Phase 4 - Hardening (3–4 days)
- Add 2+ more source adapters.
- Error handling, retries, monitoring, alerts.
- Performance tuning and test coverage.

### Phase 5 - Deploy + Demo (1–2 days)
- Deploy API/web/scraper.
- Seed sample data + create project demo script.

## 10) Testing Strategy
- **Scraper**: adapter parser unit tests with saved HTML fixtures.
- **API**: integration tests for products/history/watchlist endpoints.
- **UI**: component tests for filters/charts; e2e smoke tests for dashboard.
- **Load checks**: ingestion throughput and WebSocket fan-out sanity tests.

## 11) MVP Backlog (Prioritized)
1. Define product canonicalization rules to unify cross-source listings.
2. Build first adapter + parser fixtures.
3. Create ingestion API and `price_points` persistence.
4. Implement `/products` and `/products/:id/history`.
5. Build dashboard page with trend chart.
6. Add watchlist CRUD and alert evaluation logic.
7. Add WebSocket `price_update` events.
8. Add source health/status tracking.
9. Add auth and user settings.
10. Add two more adapters and reliability tuning.

## 12) Key Risks and Mitigations
- **Anti-bot and HTML volatility** -> adapter abstraction, robust selectors, fixture-based parser tests.
- **Duplicate/dirty data** -> canonical keys, normalization layer, dedupe hashes.
- **Realtime complexity** -> start with polling fallback, then enable WebSocket deltas.
- **Scraper outages** -> heartbeat dashboard + source-level failure alerts.
