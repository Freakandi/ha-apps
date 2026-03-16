# PP Reader

A standalone Home Assistant App that reads [Portfolio Performance](https://www.portfolio-performance.info/) `.portfolio` files and serves an interactive financial dashboard. Built as a Docker container backed by PostgreSQL, it replaces the legacy HACS integration with a modern full-stack architecture.

## Key Features

- **Automatic file watching** -- Monitors your `.portfolio` protobuf file for changes and re-ingests automatically.
- **Accurate metrics** -- TWR, IRR, FIFO cost basis, capital gains, and dividend tracking ported from Portfolio Performance's Java reference implementation.
- **Live price enrichment** -- Fetches real-time quotes from Yahoo Finance and FX rates from the ECB.
- **Interactive dashboard** -- Lit web component frontend with portfolio overview, security drill-down, time-series charts, and trade history.
- **Real-time updates** -- Server-Sent Events push data changes to the browser as they happen.
- **Home Assistant integration** -- Runs as an HA App with ingress support, sidebar panel, and HA theme compatibility.

## Quick Start

### Prerequisites

- Docker and Docker Compose
- A `.portfolio` file from Portfolio Performance desktop app

### 1. Clone and configure

```bash
git clone https://github.com/Freakandi/pp-reader-app.git
cd pp-reader-app

# Copy the example env file and edit it
cp docker/.env.example docker/.env
```

Edit `docker/.env` and set `PORTFOLIO_PATH` to the path of your `.portfolio` file inside the container (e.g., `/app/portfolio_files/my_portfolio.portfolio`) and `PORTFOLIO_DIR` to the host directory containing that file.

### 2. Start the application

```bash
docker compose -f docker/docker-compose.yml --env-file docker/.env up -d
```

### 3. Verify it's running

```bash
# Health check
curl http://localhost:8000/healthz
# Expected: {"status":"ok"}

# Dashboard data
curl http://localhost:8000/api/status
# Expected: {"last_file_update":...,"pipeline_status":"idle","version":"0.1.0"}
```

### 4. Open the dashboard

Navigate to [http://localhost:8000](http://localhost:8000) in your browser.

## Architecture Overview

PP Reader is a three-tier application running in a single Docker container:

```
┌──────────────────────────────────────────────────────┐
│                   Docker Container                    │
│                                                      │
│   FastAPI/Uvicorn    Background Scheduler   Vite SPA │
│   (REST + SSE)       (file watch, enrich,   (Lit     │
│                       metrics, sync)         components)│
│         │                    │                        │
│         └────────┬───────────┘                        │
│              asyncpg pool                             │
└──────────────────┼───────────────────────────────────┘
                   │
            ┌──────┴──────┐        ┌────────────────┐
            │ PostgreSQL  │        │ .portfolio file │
            └─────────────┘        └────────────────┘
```

**Data pipeline stages:** FileWatcher → Ingestion → CanonicalSync → Enrichment (FX + Prices) → Metrics → Snapshots

For detailed architecture decisions, see [ARCHITECTURE.md](ARCHITECTURE.md).

## Project Structure

```
pp-reader-app/
├── backend/                  # Python FastAPI backend
│   ├── app/
│   │   ├── api/              # REST endpoints and SSE
│   │   ├── db/               # Database pool, queries, migrations
│   │   ├── enrichment/       # FX rates (ECB) and prices (Yahoo)
│   │   ├── generated/        # Protobuf-generated code
│   │   ├── metrics/          # TWR, IRR, cost basis, gains, dividends
│   │   ├── models/           # Domain models and constants
│   │   └── pipeline/         # File watcher, parser, ingestion, sync
│   ├── tests/                # pytest test suite
│   └── pyproject.toml
├── frontend/                 # TypeScript Lit web components
│   ├── src/
│   │   ├── api/              # API client and realtime SSE
│   │   ├── components/       # Reusable UI components
│   │   ├── controllers/      # Lit ReactiveControllers
│   │   ├── tabs/             # Page views (overview, security, trades...)
│   │   └── styles/           # CSS with HA theme variable support
│   ├── package.json
│   └── vite.config.ts
├── docker/                   # Docker deployment
│   ├── Dockerfile            # Multi-stage build
│   ├── docker-compose.yml    # App + PostgreSQL
│   ├── run.sh                # Container entrypoint
│   └── .env.example          # Environment variable reference
├── pp_reader/                # Home Assistant add-on manifest
│   └── config.yaml           # HA App config (version auto-bumped by CI)
├── proto/                    # Portfolio Performance protobuf schema
│   └── client.proto
├── _legacy_v1/               # Legacy HACS integration (reference only)
└── pp_reference/             # PP Java reference code (reference only)
```

## Documentation

| Document | Description |
|----------|-------------|
| [ARCHITECTURE.md](ARCHITECTURE.md) | Technical design decisions and system diagram |
| [DEVELOPMENT.md](DEVELOPMENT.md) | Local development setup, testing, and workflow |
| [API.md](API.md) | REST API endpoint reference |
| [DEPLOYMENT.md](DEPLOYMENT.md) | Docker deployment and Home Assistant integration |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution guidelines and PR process |
| [EXECUTION_PLAN.md](EXECUTION_PLAN.md) | Rebuild phases and progress tracking |
| [DOCUMENTATION_MAINTENANCE.md](DOCUMENTATION_MAINTENANCE.md) | Documentation update guidelines |

## Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Backend | Python 3.13+, FastAPI, Uvicorn | REST API and async pipeline |
| Database | PostgreSQL 16, asyncpg, Alembic | Persistence and migrations |
| Frontend | TypeScript, Lit 3, Vite | Web component SPA |
| Data | protobuf, yahooquery, aiohttp | File parsing, price/FX enrichment |
| Deployment | Docker, Docker Compose | Containerized runtime |

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
