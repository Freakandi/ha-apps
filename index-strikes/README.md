# Index Strikes

Stock index options strike price recommendations based on historical volatility analysis.

A Home Assistant add-on that fetches daily OHLC data for major stock indices, calculates
rolling quantile-based strike prices, and presents an interactive dashboard with breach
detection and weekday volatility analysis.

## What it does

- **Daily strike prices** — calculates upper (call) and lower (put) boundaries using
  30-day rolling quantiles of historical returns, applied to the previous close
- **Breach detection** — tracks intraday and close breaches against calculated strikes
- **Weekday volatility** — day-of-week return comparison (10Y history vs. last 90 days)
- **Interactive dashboard** — per-index charts, statistics tables, and historical data
- **Home Assistant integration** — runs as a native add-on with ingress (sidebar panel)

## Supported indices

| Index | Ticker |
|-------|--------|
| DAX | `^GDAXI` |
| Nasdaq 100 | `^NDX` |
| S&P 500 | `^GSPC` |
| Euro Stoxx 50 | `^STOXX50E` |
| SMI | `^SSMI` |
| CAC 40 | `^FCHI` |

Additional indices can be added via the **Manage Indices** page using any Yahoo Finance ticker.

## Quick Start

### Home Assistant add-on

1. Go to **Settings → Add-ons → Add-on Store → ⋮ → Repositories**
2. Add: `https://github.com/Freakandi/index-strikes`
3. Install **Index Strikes**, configure the database options, and start the add-on.

See [DOCS.md](DOCS.md) for full installation and configuration instructions.

### Local development (Docker Compose)

```bash
cd docker
cp .env.example .env        # fill in DB credentials
docker compose up
```

The app is available at `http://localhost:8099`.

See [DEVELOPMENT.md](DEVELOPMENT.md) for the full developer workflow.

## Architecture

```
Browser  ──HTMX──▶  FastAPI (port 8099)
                       │
                  Jinja2 templates + Pico CSS
                       │
                  SQLAlchemy (async)  ──▶  PostgreSQL
                       │
                  APScheduler  ──▶  yfinance (daily fetch)
```

- **Backend:** Python 3.12, FastAPI, SQLAlchemy 2.x (async), Alembic, APScheduler
- **Frontend:** Jinja2 templates, HTMX, Pico CSS — no build step required
- **Database:** PostgreSQL (external); schema `index_strikes`
- **Data source:** Yahoo Finance via `yfinance` (daily fetch, ~10 years of history)
- **Deployment:** Docker container, HA add-on (aarch64 + amd64), ingress-ready

## Project structure

```
index_strikes/          # Python package — FastAPI app, models, services
  routers/              # API and page route handlers
  services/             # Business logic (statistics, strikes, market data)
  templates/            # Jinja2 HTML templates
  static/               # CSS, JS assets
alembic/                # Database migrations
tests/                  # pytest test suite
scripts/                # Utility scripts (dev, deploy, container helpers)
docker/                 # docker-compose.yml and .env.example for local dev
translations/           # HA add-on UI translations (en.yaml, de.yaml)
config.yaml             # HA add-on manifest
Dockerfile              # Container image definition
run.sh                  # HA add-on entrypoint (bashio)
```

## Technology stack

| Component | Technology |
|-----------|-----------|
| Language | Python 3.12+ |
| Web framework | FastAPI |
| ORM | SQLAlchemy 2.x (async) |
| Migrations | Alembic |
| Scheduler | APScheduler |
| Data source | yfinance |
| Templating | Jinja2 |
| UI interaction | HTMX |
| CSS | Pico CSS |
| Database | PostgreSQL |

## Documentation

| Document | Description |
|----------|-------------|
| [DOCS.md](DOCS.md) | HA add-on installation, configuration, and troubleshooting |
| [DEVELOPMENT.md](DEVELOPMENT.md) | Developer setup, workflow, Docker, migrations |
| [TESTING.md](TESTING.md) | Test strategy, running tests, writing new tests |
| [DEPLOYMENT.md](DEPLOYMENT.md) | Docker, HA add-on, and Raspberry Pi deployment |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution workflow, code conventions, PR process |
| [CHANGELOG.md](CHANGELOG.md) | Version history |
| [DOCUMENTATION_MAINTENANCE.md](DOCUMENTATION_MAINTENANCE.md) | Doc inventory and update triggers |

## License

See repository for license details.
