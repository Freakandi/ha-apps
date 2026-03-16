# Changelog

All notable changes to PP Reader are documented in this file.

This project uses [Semantic Versioning](https://semver.org/). The format is based on [Keep a Changelog](https://keepachangelog.com/).

---

## [Unreleased]

### Added
- Full project rebuild as a standalone HA App (Docker container)
- FastAPI backend with async PostgreSQL via asyncpg
- Lit web component frontend with Vite build
- Pipeline scheduler with file watcher and periodic enrichment
- ECB exchange rate fetcher (SDMX-JSON API)
- Yahoo Finance live price and historical price backfill
- Metrics engine aligned with Portfolio Performance Java reference:
  - TWR (Time-Weighted Return) via PerformanceIndex
  - IRR (Internal Rate of Return)
  - FIFO and moving average cost basis
  - Capital gains (realized and unrealized)
  - Dividend tracking
- REST API with 13 endpoints (dashboard, portfolios, accounts, securities, trades, wealth, performance, status)
- Server-Sent Events (SSE) for real-time push updates
- Alembic database migrations
- Protobuf `.portfolio` file parser
- Docker multi-stage build with cache-busting via `BUILD_VERSION` ARG
- Home Assistant App manifest (`pp_reader/config.yaml`) with ingress support
- HA add-on repository layout (`repository.yaml`, `pp_reader/config.yaml`)
- GitHub Actions release pipeline for multi-arch Docker builds and GHCR publishing
- Comprehensive test suites (pytest for backend, vitest for frontend)
- Full project documentation suite

### Changed
- Migrated from Home Assistant HACS integration to standalone Docker App
- Replaced SQLite with PostgreSQL
- Replaced sync Python with async-native architecture
- Replaced WebSocket-only API with REST + SSE
- Replaced vanilla DOM manipulation with Lit web components
- Replaced module-scoped global stores with ReactiveController pattern

---

## [0.1.0] - 2026-03-14

Initial rebuild release. Complete rewrite of the legacy v1 HACS integration.

See [EXECUTION_PLAN.md](EXECUTION_PLAN.md) for the full rebuild phase breakdown.

---

## Versioning Policy

PP Reader follows semantic versioning:

- **MAJOR** (1.0.0): Breaking API changes, incompatible database migrations
- **MINOR** (0.x.0): New features, new endpoints, non-breaking schema changes
- **PATCH** (0.0.x): Bug fixes, performance improvements, documentation updates

### How to Add a Changelog Entry

When preparing a release:

1. Move items from `[Unreleased]` to a new version section with the release date.
2. Group changes under the appropriate heading:
   - **Added** -- New features
   - **Changed** -- Changes to existing functionality
   - **Deprecated** -- Features that will be removed
   - **Removed** -- Features that were removed
   - **Fixed** -- Bug fixes
   - **Security** -- Vulnerability fixes
3. Keep entries concise (one line per change).
4. Reference related issues or PRs where applicable.
5. Push a version tag (e.g., `git tag v0.2.0 && git push origin v0.2.0`). The release pipeline builds Docker images and auto-bumps `pp_reader/config.yaml`. See [DEVELOPMENT.md](DEVELOPMENT.md) → "Releasing a Version".

### Example Entry

```markdown
## [0.2.0] - 2026-04-15

### Added
- WebSocket support for bidirectional real-time filtering (#42)
- Dark mode theme toggle in settings

### Fixed
- FX rate lookup returns stale data for weekend dates (#38)
```
