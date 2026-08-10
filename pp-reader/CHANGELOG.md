# Changelog

All notable changes to PP Reader are documented in this file.

This project uses [Semantic Versioning](https://semver.org/). The format is based on [Keep a Changelog](https://keepachangelog.com/).

---

## [Unreleased]

---

## [0.1.17] - 2026-08-10

### Added
- The "Transaktionen" (Trades) tab now shows every booking from your file
  that has a security attached — Sollzins, Steuer, Steuererstattung,
  Gebühr and Gebührenerstattung bookings used to be missing from the list
  entirely (only Kauf, Verkauf, Ein-/Auslieferung, Depotübertrag, Dividende
  and Zins showed up). A new "Buchungsart" filter row above the list lets
  you show or hide individual booking types, so the longer list stays easy
  to scan. The "Gebühren" column now shows the sum of all fee portions on
  a booking instead of silently dropping every portion after the first.
- The "Zeitmaschine" tab now shows real numbers for "Zeitgew. Rendite (TWR)"
  and "Int. Zinsfuß (IRR)" — previously these two tiles were always blank.
  Changing the date range now updates every tile below the chart, including
  "Gewinn / Verlust": deposits and withdrawals in the selected period are no
  longer counted as gain or loss, only the actual change in value is (a
  deposit shortly before the end of the range used to inflate "Gewinn /
  Verlust" as if it were investment growth). Days with a missing price
  (the same gap days the chart already breaks the line at) are now left
  out of these calculations entirely, instead of counting a partial,
  incomplete value as if it were the day's real wealth — which used to be
  able to swing "Zeitgew. Rendite (TWR)" wildly (e.g. to -100%) right next
  to a correct "Gewinn / Verlust" on the same card.
- The "Zeitmaschine" tab's wealth chart now shows your full history since
  the first transaction you ever booked, instead of starting on the day the
  add-on happened to first run — and it survives a database rebuild instead
  of coming back empty. Days where a price is missing for one or more
  positions (beyond a short grace period for normal weekend/holiday gaps)
  now show as a visible break in the line, instead of a straight line
  papering over the gap, with a note below the chart naming how many days
  and up to how many positions are affected — hovering near such a day now
  snaps to the nearest day with a real value instead of showing that day's
  incomplete partial amount. A new "Seit Beginn" button in the date-range
  picker jumps straight to that full history in one click.
- `GET /api/status` now reports how many bookings were read from your file
  and how many were processed on the last sync, plus a per-type breakdown
  (`transactions_read`, `transactions_processed`,
  `transaction_type_breakdown`, `ignored_transaction_types`) — if a booking
  type were ever left unprocessed, it would be named there directly instead
  of only showing up as a smaller total.

### Changed
- Steuer and Steuererstattung bookings in the "Transaktionen" tab and on a
  security's detail page now always show "—" under "Anteile" and "Kurs",
  instead of sometimes showing a real-looking number there. These bookings
  don't represent buying or selling shares, so the "share count"/"price"
  the file records for them is really a tax rate — showing it next to real
  trade rows read as if it were a market price. Underlying data is
  unchanged; this is a display-only rule.
- Documented a startup risk for existing users, present since 0.1.16: if the
  add-on no longer starts after an update and stays in the "stopped" state,
  with nothing shown in the app and nothing in its own log, the likely cause
  is a `portfolio_path` value saved before 0.1.16 that no longer matches the
  pattern enforced since then (`/share/`, `/config/`, or `/media/`, ending in
  `.portfolio`). Open the add-on's Configuration tab, re-check `portfolio_path`,
  and save it again — an invalid value is then flagged directly instead of
  silently blocking the start. No code or validation behavior changed; see
  the Store page README for the full explanation.

### Fixed
- The "Zeitmaschine" tab's wealth curve no longer drops by roughly 100.000 €
  in early December 2025 and jumps back up on 1 June 2026 — money you move
  into an account you have since marked as "stillgelegt" (retired) in
  PortfolioPerformance is no longer treated as if it had left your assets.
  Only the current asset list hides retired accounts and depots; the history
  behind the chart was hiding them too, so 98.000 € parked in a retired
  account for half a year simply disappeared from the curve for that whole
  period, and reappeared out of nowhere when it was transferred back.
  Because of this, the "1 Jahr" view reported a **loss** of −38,95 %
  (Zeitgew. Rendite) where PortfolioPerformance shows +13,27 % for the same
  file; it now reports +14,07 % (Int. Zinsfuß +13,87 % vs. PP's +13,22 %).
  The remaining difference of well under one percentage point comes from
  three papers your price provider does not deliver quotes for (below) and
  from the two programs reading live prices at different moments; the
  purchase values ("Kaufwert") were never affected and are unchanged.
- The "Zeitmaschine" tab now really calculates the period you asked for. A
  handful of papers that the price provider does not list at all — in your
  file two long-dated options — used to disqualify **every single day** they
  were held, and those days were then silently left out of the calculation:
  the "1 Jahr" view was in fact computed over 09.08.2025–12.02.2026 only,
  six months short, with nothing on screen saying so. Now a day is only
  discarded when nothing about it can be established at all; a paper whose
  latest quote is merely old is valued at that last real quote (never an
  invented or interpolated price), and a paper with no quote at all counts
  as 0 € for that day. Two things make the remainder visible instead of
  silent: the note below the chart now names how many days are affected and
  up to how many positions were counted as 0 €, and the "Performance" card
  now states in plain words which period was actually calculated — either
  "Gerechnet über den vollen Zeitraum …" or, if it had to be shortened,
  both the calculated and the requested period.
- "Gewinn/Verlust" for a position that was ever moved between two depots
  (Depotübertrag) is now shown correctly in the depot it landed in —
  previously it showed the full market value as the "gain" (e.g. +137.69%
  instead of the correct +37.69% for an Amazon position moved on
  2026-08-04), because the purchase value carried over by the transfer was
  silently dropped from the gain calculation for that depot. This also
  inflated the affected depot's own "Gesamt +/-" row and the overview's
  "Summe" row by the same amount for every transferred position they
  contain. Holdings, purchase value ("Kaufwert") and the wealth-over-time
  chart were never affected — only the gain figure itself.
- A security's detail card now shows gains in green and losses in red
  again — on every metric that can trend (Gewinn/Verlust, Tagesveränderung,
  Marktwert), both the native-currency line and its EUR-equivalent line
  underneath. Every value on the card previously rendered in the plain
  text color regardless of sign, because the trend color rule and the
  card's own text-color rule applied to the exact same element with the
  same specificity weight, and the card's rule happened to win — a defect
  present since April 2026, invisible until closely compared against the
  mockup.
- The "Zeitmaschine" tab's date-range picker now opens as a properly
  styled calendar (two-month grid, weekday headers, preset sidebar,
  footer) instead of a giant unstyled block of native-looking controls
  with the weekday initials run together as plain text and the day
  numbers stacked one per row — almost all of the picker's CSS was
  missing, only the closed trigger button had ever been styled. Clicking
  a preset (e.g. "Seit Beginn") or a calendar day always changed the
  selection correctly under the hood; the missing styling just made it
  look like nothing happened. Also fixed: reopening the picker after
  applying a range no longer fails to highlight the previously-selected
  start/end day outside UTC — the applied range was round-tripped through
  a UTC-parsed date string, `getTimezoneOffset()` away from the local day
  the calendar actually needed.
- Cards throughout the app (e.g. the overview's depot and account cards)
  now sit close together (6px gap) instead of far apart (24px) — matching
  the design mockups, which have shown 6px since an April 2026 design
  rework that the shared card stylesheet never picked up.
- Einlieferung and Auslieferung bookings now show their "Typ" in green resp.
  red, matching every other booking type — both in the "Transaktionen" tab
  (also "Betrag" there) and in a security's own transaction list on its
  detail page, where Einlieferung bookings previously also showed the raw
  English booking-type name instead of "Einlieferung". Both places showed up
  colorless (neutral) and/or in English because of a leftover key mismatch
  in the trend-color/label lookup.
- The overview's "Gesamt +/-" and "%" figures for a depot are now consistent
  with the positions inside it: the depot's total gain/loss is now the sum
  of its positions' gains (matching what the position rows and the "Summe"
  footer already showed), instead of a separately computed market-value-
  minus-purchase-value figure that could show a different number right next
  to the same total. The "%" next to "Gesamt +/-" now matches "Gesamt +/- ÷
  Kaufwert" wherever a Kaufwert is shown — previously it used a different,
  hidden cost basis (all-time invested amount, not reduced by sales), which
  could show a misleadingly small percentage for any position that had been
  partially sold. Where no Kaufwert is shown (a zero purchase value), no
  percentage is shown either — unchanged.
- The add-on no longer aborts on startup if its database is temporarily
  unreachable (for example, an external database that is down or not
  reachable over the network). It now starts normally and reports the
  problem in a machine-readable way via `/api/status` (new `db_error` /
  `pipeline_error` fields), instead of crashing outright. It automatically
  retries the database connection every 30 seconds and recovers on its own
  once the database is reachable again — no restart needed. While the
  database is unreachable, the data endpoints (dashboard, portfolios,
  accounts, securities, trades, wealth, performance) respond with a clear
  error status instead of hanging or crashing.
- The database-unreachable and pipeline-start-failed states are now also
  shown directly in the web interface as a clear error banner (instead of
  only being reported via `/api/status` for machine consumers) — visible on
  every tab while the problem persists, and disappearing on its own again
  once the add-on self-heals.
- Depot-to-depot transfers (moving a position from one depot to another in
  Portfolio Performance) are now processed correctly. Previously the
  receiving depot did not show the transferred position at all (and so it
  was also missing from that depot's totals), while the giving depot kept
  showing it as if nothing had moved. The moved shares now keep their
  original purchase cost when they change depots (no revaluation), the
  transfer now appears as its own row in the "Transaktionen" tab, and the total
  wealth tile is unaffected by a transfer, as expected. Buying and selling
  the same security on the same day now also gives the same purchase cost
  and gain regardless of the order the bookings appear in the imported file
  — previously the file's row order could change the result. If more
  shares are ever sold or transferred than are actually held for a
  position, this is now recorded as a warning in the add-on's log instead
  of being silently ignored. Transferring more shares out of a depot than
  it actually holds is now also recorded as a warning, the same way.
- If you hold the same security in more than one depot, each depot's own
  row (purchase cost, position size, gain) now shows only what that depot
  itself holds — previously it showed the combined total of every depot
  holding that security, so the numbers looked too high (and matched
  nothing you could check by hand). Adding up the depot rows now gives
  exactly the total the security's own detail view shows, which now also
  breaks the total down per depot. Dividends, taxes and fees booked on a
  security you hold in several depots are now counted exactly once — in
  the depot whose settlement account they were paid to — instead of once
  per depot; if none of your depots' settlement accounts match, the
  booking is now shown as "not assigned" in the add-on's internal run data
  instead of silently disappearing or being spread across depots.
- "Ø Kaufpreis" (average purchase price) now always comes from the same
  calculation as "Kaufwert" (purchase value) — previously the two were
  computed independently and could drift apart, so "Ø Kaufpreis × Bestand"
  did not always add up to "Kaufwert" in the matching currency. If you hold
  a position in a foreign currency and its purchase price truly cannot be
  determined in that currency (no matching entry in the imported file at
  all — an edge case that did not occur in a full check against real
  portfolio data), "Ø Kaufpreis" is now shown as "—" instead of a number
  that was actually in EUR but labeled with the foreign currency. The
  security's own detail view now also shows the EUR-equivalent average
  purchase price alongside the foreign-currency one (matching the position
  row in the overview, which already showed both), so "Ø Kaufpreis ×
  Bestand = Kaufwert" is checkable there too, in EUR.
- The security detail card's "Tagesveränderung" (day change) tile showed
  a EUR amount labelled with the security's own foreign currency (e.g. a
  KRW position showed a EUR number tagged "KRW") — it now shows the
  correct native-currency day change on top, with the EUR amount on its
  own line underneath. The same native-currency-on-top / EUR-underneath
  layout now also applies to "Kurs" (quote date moved into parentheses
  after the price, EUR-equivalent added below for foreign-currency
  positions), "Kaufwert", "Marktwert" and "Gewinn/Verlust" — the last one
  shows its own percentage change on each of the two lines, since currency
  movement between the purchase date and today can legitimately make the
  native-currency and EUR percentages differ. "Kaufwert" in EUR is now
  converted using the exchange rate on each purchase lot's own date (or
  the closest earlier rate if none exists for that exact day), summed
  across lots for positions bought in several batches — instead of always
  using today's rate. For a security already priced in EUR, all of the
  above collapse back to a single line, same as before. Affects the
  security detail card only; does not change any portfolio-level totals.

### Security
- Hardened the add-on start script (`docker/run.sh`) against inherited shell
  tracing: it now disables `xtrace` (`set +x`) as its very first action, so
  the DB password and Supervisor token it processes no longer appear in
  xtrace output.
- Removed the real portfolio file `config/pp_reader_data/S-Depot.portfolio`
  (actual financial data) from git tracking — the file stays local only.
  `.gitignore` now blocks `*.portfolio` globally so no portfolio data file
  can be committed anywhere in the repo. A tracked
  `config/pp_reader_data/README.md` documents the local-only convention
  (`S-Depot-JJJJ-MM.portfolio`, access via path configuration only). Note:
  previously pushed versions remain in the git history on GitHub (private
  repo); a full history purge was deliberately not performed and would be a
  separate decision.

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
