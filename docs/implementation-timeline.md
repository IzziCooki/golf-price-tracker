# Golf Price Tracker - Step-by-Step Implementation Timeline

## Week 1 - Foundation + First Data Flow

### Day 1: Project Bootstrap
1. Initialize monorepo structure (`api`, `scraper`, `web`, `shared`, `infra`, `docs`).
2. Add root `.gitignore`, `.env.example`, and basic README.
3. Set up Docker Compose for MongoDB and local services.
4. Define shared environment variables and naming conventions.

### Day 2: API Core Setup
1. Initialize Express app with TypeScript (or JS if preferred).
2. Add MongoDB connection and health check route (`/health`).
3. Add global middleware (CORS, JSON parsing, request logging).
4. Add error-handling middleware and consistent API response format.

### Day 3: Database Models
1. Create Mongoose models: `Product`, `Source`, `Listing`, `PricePoint`.
2. Add indexes for `PricePoint` (`productId + capturedAt`, `listingId + capturedAt`).
3. Implement canonical product key logic for dedupe/normalization.
4. Add seed script with a small set of golf products and sources.

### Day 4: Scraper Core Setup
1. Initialize Python scraper service with virtual environment and dependency file.
2. Build `BaseAdapter` interface and scraper runner.
3. Add HTTP client, retry policy, and per-domain rate limiting.
4. Add structured logs and error categorization by source.

### Day 5: First Source Adapter
1. Implement adapter for one target source.
2. Parse listing fields (name, price, URL, stock, SKU).
3. Normalize parsed records into canonical schema.
4. Save HTML fixtures and parser tests for stability.

### Day 6: Ingestion Path
1. Build internal ingestion endpoint (`POST /api/ingest/price-point`).
2. Add ingestion auth key validation.
3. Upsert products/listings and insert normalized price points.
4. Add idempotency guard to prevent duplicate inserts.

### Day 7: End-to-End Validation
1. Run scraper against sample products.
2. Confirm data appears in Mongo collections correctly.
3. Validate dedupe, normalization, and failure handling.
4. Document first full data flow in `docs`.

---

## Week 2 - Analytics API + Realtime

### Day 8: Product Retrieval Endpoints
1. Build `GET /api/products` with filters (`category`, `brand`, `q`).
2. Add pagination/sorting.
3. Return latest known price per product.

### Day 9: History + Trend Endpoints
1. Build `GET /api/products/:id/history`.
2. Add time range support (`7d`, `30d`, `90d`).
3. Compute basic metrics (min, max, avg, delta).

### Day 10: Analytics Endpoints
1. Build `GET /api/analytics/top-movers`.
2. Add volatility and percentage drop calculations.
3. Add source freshness analytics for health panel.

### Day 11: Realtime Channel
1. Set up WebSocket server (or Socket.IO).
2. Emit `price_update` on new `PricePoint`.
3. Emit `source_status` when source health changes.

### Day 12: Polling Fallback
1. Add polling endpoint for clients that cannot use WebSockets.
2. Implement delta-based feed (`since` timestamp).
3. Ensure same payload shape as WebSocket events.

### Day 13: API Reliability
1. Add API rate limiting.
2. Add request IDs for traceability.
3. Tighten validation with Zod/Joi for request payloads.

### Day 14: API Test Pass
1. Add integration tests for products/history/analytics/ingest.
2. Add tests for bad input/auth failure paths.
3. Fix edge cases discovered by test runs.

---

## Week 3 - React Dashboard MVP

### Day 15: Frontend App Setup
1. Initialize React app with Tailwind.
2. Add app layout, nav, and route skeletons.
3. Configure API client + React Query.

### Day 16: Dashboard Overview Page
1. Create KPI cards (total products, biggest drops, avg movement).
2. Add top movers table.
3. Add source freshness/status panel.

### Day 17: Product Detail Page
1. Add product search/select flow.
2. Build trend line chart (Recharts/Chart.js).
3. Show min/max/avg and source price comparison.

### Day 18: Filters + URL State
1. Add category/brand/time-range filters.
2. Persist filters in URL query params.
3. Sync filter changes with React Query cache keys.

### Day 19: Watchlist MVP
1. Build watchlist UI and CRUD actions.
2. Add target price input and per-item controls.
3. Show current vs target with status badges.

### Day 20: Live Feed Integration
1. Connect WebSocket client.
2. Merge incremental updates into existing state.
3. Display live activity feed and update timestamps.

### Day 21: Frontend Polish
1. Add loading/empty/error states across pages.
2. Improve responsiveness for mobile and tablet.
3. Tune chart readability and table sorting behavior.

---

## Week 4 - User Features, Hardening, and Launch

### Day 22: Auth + User Settings
1. Implement JWT auth (register/login).
2. Protect watchlist endpoints.
3. Add user preferences (default filters, currency, alert settings).

### Day 23: Alerts Engine
1. Implement alert evaluation on incoming price points.
2. Trigger `price_drop_alert` events.
3. Persist alert history (`alerts` collection).

### Day 24: Additional Source Adapters
1. Add second source adapter.
2. Add third source adapter.
3. Extend normalization mappings across adapter differences.

### Day 25: Scraper Scheduling + Recovery
1. Add scheduler (APScheduler/Celery beat).
2. Set high-priority and full-crawl schedules.
3. Add circuit breaker for repeatedly failing sources.

### Day 26: Observability + Ops
1. Add service-level logs for API and scraper.
2. Add scraper heartbeat and stale-source detection.
3. Add monitoring notes/runbook in `docs`.

### Day 27: Final Test and Performance Pass
1. Run full API + scraper + UI test suite.
2. Perform ingestion throughput sanity checks.
3. Optimize slow queries and indexes if needed.

### Day 28: Deployment + Demo Prep
1. Deploy API, scraper worker, and frontend.
2. Configure production environment variables/secrets.
3. Seed demo data and capture screenshots/GIF walkthrough.
4. Finalize README with architecture diagram and setup steps.

---

## MVP Exit Checklist
- At least 3 sources scraping successfully.
- Historical trends visible per product.
- Live updates working via WebSocket (or polling fallback).
- Watchlist and target-price alerts functional.
- Core endpoints documented and tested.
- Deployable environment and demo-ready UI complete.
