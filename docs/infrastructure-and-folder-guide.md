# Infrastructure and Folder Guide

This project uses a **monorepo** layout with separate services for scraping, API, and frontend UI.  
The system flow is:

1. `scraper` collects and normalizes golf equipment price data.
2. `api` stores/query data in MongoDB and serves REST/WebSocket updates.
3. `web` renders dashboards, charts, filters, and watchlists.
4. `shared` keeps cross-service types/constants consistent.
5. `infra` contains deployment/runtime support files.

---

## Top-Level Folders

### `api/`
Node.js/Express backend for ingestion, product/history endpoints, analytics, auth, and realtime updates.

- `src/`
  - `config/` - environment loading, DB config, app-level settings.
  - `controllers/` - request handlers for API routes.
  - `middleware/` - auth, validation, error handling, request logging.
  - `models/` - MongoDB/Mongoose schemas (`Product`, `Listing`, `PricePoint`, etc.).
  - `routes/` - route definitions and route grouping by domain.
  - `services/` - business logic (analytics, alert evaluation, ingestion processing).
  - `utils/` - shared backend helpers (formatting, constants, utility functions).
- `tests/` - integration and endpoint-level API tests.

### `scraper/`
Python automation service that fetches source pages/APIs, parses data, normalizes output, and sends to API ingestion.

- `src/`
  - `adapters/` - one adapter per source site/provider.
  - `core/` - runner, request client, retry/rate-limit logic, orchestration.
  - `normalizers/` - canonicalization and data cleanup rules.
  - `schedulers/` - cron/interval job definitions and scheduling logic.
  - `utils/` - scraper helpers (parsing utilities, hashing, logging helpers).
- `tests/`
  - `fixtures/` - saved source HTML/JSON for deterministic parser tests.
  - `unit/` - unit tests for adapters, normalizers, and core logic.

### `web/`
React + Tailwind dashboard for live and historical visualization.

- `public/` - static assets.
- `src/`
  - `components/` - reusable UI building blocks.
    - `charts/` - trend/analytics chart components.
    - `filters/` - filter controls (category, brand, date range).
    - `layout/` - app shell, nav, headers, shared layout elements.
    - `watchlist/` - watchlist cards, forms, and list components.
  - `hooks/` - custom React hooks for data and UI behavior.
  - `pages/` - route-level pages (dashboard, product detail, watchlist, etc.).
  - `services/` - API client, websocket client, request wrappers.
  - `store/` - state management setup (local/global UI state).
  - `styles/` - global styles and Tailwind-related style structure.
  - `types/` - frontend-specific TypeScript types/interfaces.
  - `utils/` - UI/data formatting helpers.

### `shared/`
Code shared across services to reduce duplication and drift.

- `types/` - shared domain contracts (price point payloads, DTOs).
- `constants/` - enums, fixed keys, category/source constants.

### `infra/`
Operational infrastructure files for local/dev/prod workflows.

- `docker/` - Dockerfiles and Compose-related setup.
- `monitoring/` - health checks, observability config, alerting notes.
- `scripts/` - automation scripts (bootstrap, seed, deploy helpers).

### `docs/`
Project documentation: planning, architecture, implementation timelines, and operational guides.

---

## How the Infrastructure Works Together

1. **Scrape cycle starts** in `scraper/src/schedulers`.
2. **Adapters fetch and parse** source data in `scraper/src/adapters`.
3. **Normalizer standardizes** product/price payloads in `scraper/src/normalizers`.
4. **Ingestion endpoint receives data** in `api/src/routes` + `api/src/controllers`.
5. **Service layer persists and computes analytics** in `api/src/services` + `api/src/models`.
6. **Web app queries API** via `web/src/services` and renders views in `web/src/pages`.
7. **Live updates stream** from API to UI via websocket events for price changes/alerts.

This structure keeps responsibilities isolated, makes scaling easier (each service can scale independently), and supports clean testing boundaries.
