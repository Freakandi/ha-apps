# Changelog

All notable changes to PP Reader are documented in this file.

This project uses [Semantic Versioning](https://semver.org/). The format is based on [Keep a Changelog](https://keepachangelog.com/).

---

## [Unreleased]

---

## [0.1.16] - 2026-07-27

This is a collective section: it bundles everything not yet documented in a
dedicated changelog section since `[0.1.0]`. Versions 0.1.1-0.1.15 were built
and released without their own changelog sections, so this section mixes two
kinds of entries: changes already shipped in 0.1.1-0.1.15 that were never
written down at the time, and changes that are genuinely new in 0.1.16 —
among the latter: the visible configuration-error message for an unset,
example, or typo'd `portfolio_path`, the Supervisor path validation and its
field description, the new Add-on Store page, the fixed homepage link, the
version now shown at `/api/status`, and automatic resume when a missing
`.portfolio` file reappears. Also new for anyone on 0.1.15: all fixes that
landed after that build without a release, most notably the currency
(FX) conversion corrections for securities, accounts, and day-change
values.

### Added
- "Letzter Kurs" column in expanded position table showing the latest price in
  native currency.
- `pp_reader/translations/en.yaml` with a field description for the Home
  Assistant Supervisor's Configuration form, marking the shipped
  `portfolio_path` default as an example path to replace, not a real file.

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
- The app now reports the Home Assistant add-on version at `/api/status` instead
  of a fixed `0.1.0`. `pp_reader/config.yaml` is the single source of truth for
  the version; local/dev runs show it suffixed with `-dev`.
- The `version:` field in `pp_reader/config.yaml` is now read by a single,
  tested reader (`scripts/read-version`) at every build path (`make
  docker-build`/`rebuild`, the release workflow), instead of three separate
  ad-hoc parsers. An empty or malformed `version:` field now loudly fails the
  build/release instead of silently producing an empty or broken image tag.
- Migrated all raw HA CSS variables in `cards.css` to canonical `--pp-*` token
  aliases (Phase 90). Replaced ~40 HA variable usages (`--primary-text-color`,
  `--card-background-color`, `--divider-color`, etc.) and 7 hardcoded `rgba()`
  values with tokens defined in `theme.css`. Stripped redundant token fallbacks
  (`--pp-flash,`, `--pp-chart-baseline,`, `--pp-bg-secondary,`). Aligned
  zebra-stripe and hover selectors to target `td` per design system §4.6.4,
  adding missing even-row rule.


- Price charts now auto-scale the y-axis to the visible data range for the selected
  period, with 5% padding. The baseline (Einstandskurs) line is hidden when it falls
  outside the visible range. This makes short-period charts (1M, 6M) as informative
  as long-period ones.
- Split `<pp-header-card>` into `<pp-app-bar>` (fixed title bar) and
  `<pp-header-card>` (sticky tab navigation header). Removes class-based
  CSS positioning hacks. Adds `--app-bar-background-color` /
  `--app-bar-text-color` CSS variables for future primary bar theming.
- Migrated from Home Assistant HACS integration to standalone Docker App
- Replaced SQLite with PostgreSQL
- Replaced sync Python with async-native architecture
- Replaced WebSocket-only API with REST + SSE
- Replaced vanilla DOM manipulation with Lit web components
- Replaced module-scoped global stores with ReactiveController pattern
- `portfolio_path` in `pp_reader/config.yaml` is now validated by the Home
  Assistant Supervisor (`match(REGEX)`) before the add-on even starts,
  rejecting an empty value and obviously malformed paths (wrong extension,
  relative path, or outside the mapped `share`/`config`/`media`
  directories). The shipped example path itself still validates as a
  well-formed path — the Supervisor schema cannot tell an edited path from
  an unedited one, only the app can (see next entry).

### Removed
- armv7 (32-bit ARM) is no longer listed as a supported architecture. No armv7
  version of this add-on was ever published, so Home Assistant will no longer
  offer installation on 32-bit ARM systems.

### Fixed
- Leaving `portfolio_path` empty (or blank) no longer leaves the add-on stuck
  reporting a pipeline error with a permanently empty dashboard. Previously
  an empty path was silently treated as the current directory, which does
  exist, so the add-on tried to parse it as a `.portfolio` file; that error
  was caught and logged, but the add-on kept reporting an error status and
  the dashboard never showed any data. Now file watching and the startup
  parse are simply disabled until a path is configured (the log says so
  once at startup). A path that doesn't exist (e.g. a typo) now also logs
  a warning instead of failing completely silently.
- An unconfigured, unedited-example, or typo'd `portfolio_path` is no longer
  a silent failure mode you can only diagnose via the log. The Overview page
  now shows a clear configuration-error message in all three cases
  after the add-on starts — no log access needed — and the message
  disappears on its own as soon as a real `.portfolio` file exists at the
  configured path (`GET /api/status`, plus a live SSE update for a
  dashboard tab already open at the moment of detection).
- The Home Assistant add-on manifest's `url:` field (`pp_reader/config.yaml`)
  now points to the public `https://github.com/Freakandi/ha-apps` instead of
  the private development repository. The Add-on Store's "Visit the …
  homepage" link previously led to a 404 for anyone without access to the
  private repo.
- The Add-on Store page now shows its own installation and configuration
  guide instead of the developer README. The previous page carried a
  `git clone` instruction for the private source repository and several
  links to files that were never published — both unusable for anyone
  installing the add-on from the store.
- Overview header meta (Gesamtvermögen / Zuletzt aktualisiert) now renders
  on two separate lines as intended (`flex-direction: column` added to
  `pp-header-card [slot="meta"]`).
- `--pp-reader-bar-height` CSS variable now correctly tracked via
  `ResizeObserver` on `<pp-app-bar>` (was: dead selector, always defaulted
  to 100px).
- Restored white background, padding and border-radius on all
  `pp-header-card` instances (regression introduced in phase 47).
- Fixed meta information and nav-dot row appearing above the tab title
  in Light DOM flex layout (CSS `order` correction).
- Tab header card now shrinks and hides meta row when stuck to the top
  of the scroll container (IntersectionObserver sentinel replaces the
  non-functional `::slotted()` selector).
- FX conversion in metrics engine: non-EUR security values (current_value)
  are now correctly divided by the ECB rate instead of using the native
  price directly as EUR. Portfolios with USD/GBP/etc. positions will now
  show accurate EUR valuations.
- Account balance FX conversion: non-EUR account balances are now correctly
  converted to EUR by dividing by the ECB rate (previously multiplied,
  overstating cash balances).
- Day-change FX conversion: `day_change_eur` for non-EUR positions is now
  computed by dividing by the ECB rate (previously multiplied).
- Transaction EUR amount enrichment: after FX rates are fetched,
  `transactions.amount_eur_cents` is now populated for non-EUR transactions,
  fixing purchase-value and gain calculations for portfolios with non-EUR
  brokerage accounts.
- Positions that were fully sold in one portfolio no longer appear as ghost entries
  in the portfolio position list.
- Portfolio-level daily change ("Heute +/-") now correctly shows the sum of position
  day-changes instead of duplicating the total gain.
- "Anz. Pos." column now counts only active positions (holdings > 0).
- Day-change enrichment now correctly scopes the UPDATE per (portfolio, security) pair,
  fixing incorrect values when the same security is held in multiple portfolios.
- Portfolio and position percentage values (Heute +/- %, Gesamt +/- %) were displayed
  100× too small due to API returning decimal fractions instead of percentage points;
  API now scales all gain_pct / day_change_pct values by ×100 before serializing to JSON.
- If the configured `.portfolio` file is missing when the add-on starts (or
  goes missing later) and is then created/restored, the dashboard now updates
  right away. Previously the add-on only noticed the file was there again and
  quietly remembered its timestamp — the actual data refresh only happened on
  the next edit to the file or after a full add-on restart.

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
5. Publish the release by following [DEVELOPMENT.md](DEVELOPMENT.md) → "Releasing a Version" (version bump on a work branch → PR to `dev` → release PR `dev` → `main`, which triggers the pipeline). There is no tag-based release trigger, and the pipeline does not bump `pp_reader/config.yaml` itself — that bump is a manual prerequisite.

### Example Entry

```markdown
## [0.2.0] - 2026-04-15

### Added
- WebSocket support for bidirectional real-time filtering (#42)
- Dark mode theme toggle in settings

### Fixed
- FX rate lookup returns stale data for weekend dates (#38)
```
