# PP Reader

A Home Assistant add-on that turns your [Portfolio Performance](https://www.portfolio-performance.info/) file into a live financial dashboard, right inside Home Assistant.

## What it does

- **Automatic updates** — checks your `.portfolio` file for changes every 60 seconds by default and refreshes the dashboard automatically, no manual re-import needed.
- **Cost basis and gain/loss** — average purchase price, purchase value, current market value, and gain/loss in absolute terms and percent, plus the day's change.
- **Live prices** — fetches current share prices and currency exchange rates on a schedule you control, and converts non-EUR holdings at the current ECB rate.
- **Interactive dashboard** — portfolio overview, per-security detail, historical charts, and trade history.
- **Real-time** — the dashboard updates itself as new data arrives.
- **Native Home Assistant integration** — appears as a sidebar panel and follows your Home Assistant theme.

PP Reader is under active development. Money-weighted return (IRR) and dividend income are already computed internally but not shown anywhere yet; time-weighted return is implemented but not yet wired into the calculation pipeline.

## Installation

1. In Home Assistant, go to **Settings → Add-ons → Add-on Store**.
2. Click the **⋮** menu (top right) → **Repositories**.
3. Add this repository URL: `https://github.com/Freakandi/ha-apps`
4. Close the dialog and refresh the store page — **PP Reader** now appears in the list.
5. Click **PP Reader**, then **Install**. Home Assistant pulls the pre-built image for your device's architecture.

## Configuration

Before starting the add-on for the first time:

1. Export or copy your `.portfolio` file (from the Portfolio Performance desktop app) into Home Assistant's `/share` directory — for example `/share/portfolios/my_portfolio.portfolio`.
2. Open the add-on's **Configuration** tab and set `portfolio_path` (labeled **Portfolio file path** in the form) to that file's path. The pre-filled value (`/share/portfolios/my_portfolio.portfolio`) is only an **example**, not a real file — if you leave it unchanged and never place a file there, the add-on shows a clear configuration-error message in the dashboard after starting (see the table below).
3. Leave `db_mode` set to `local` — the add-on runs its own PostgreSQL database automatically. Only switch it to `external` if you already run your own PostgreSQL server, and fill in `db_host`, `db_port`, `db_name`, `db_user`, and `db_password` in that case.
4. Click **Start**, then open **PP Reader** from the Home Assistant sidebar.

Full settings reference (defaults shown below). **Required?** means: do you
need to set this yourself, and what happens if you don't.

| Option | What it controls | Default | Required? |
|--------|-------------------|---------|-----------|
| `db_mode` | Whether PostgreSQL runs locally (managed by the add-on) or connects to an existing external server | `local` | No |
| `db_host` | PostgreSQL host — only used when `db_mode` is `external` | *(empty)* | Only if `db_mode` is `external` — the add-on won't start without it |
| `db_port` | PostgreSQL port — only used when `db_mode` is `external` | `5432` | No |
| `db_name` | PostgreSQL database name — only used when `db_mode` is `external` | `pp_reader` | Only if `db_mode` is `external` — a value that doesn't match a real database on that server fails startup |
| `db_user` | PostgreSQL username — only used when `db_mode` is `external` | `pp_reader` | Only if `db_mode` is `external` — a value that doesn't match a real user on that server fails startup |
| `db_password` | PostgreSQL password — only used when `db_mode` is `external` | *(empty)* | Only if `db_mode` is `external` — a wrong or empty value fails startup |
| `portfolio_path` (labeled **Portfolio file path** in the form) | Path to your `.portfolio` file inside Home Assistant. Must start with `/share/`, `/config/`, or `/media/` and end in `.portfolio` | `/share/portfolios/my_portfolio.portfolio` (**example only** — not a real file) | Yes — Home Assistant checks this value against the required format when you save it and refuses to save an empty or malformed one, so you cannot configure such a value today. (Values saved before 0.1.16, when no format was enforced, may still be stored and are what the Troubleshooting section below is about.) A correctly formatted path with no file there yet (e.g. the unchanged example, or a typo) shows a clear configuration-error message on the Overview page after starting, no log access needed; the message disappears and the add-on picks the file up automatically as soon as a real `.portfolio` file exists at the configured path; if the file exists but isn't a valid `.portfolio` file the log shows a pipeline error |
| `file_poll_interval` | How often (seconds) the add-on checks the portfolio file for changes | `60` | No |
| `enrich_interval` | How often (seconds) prices and exchange rates are refreshed | `3600` | No |
| `log_level` | Add-on log verbosity (`DEBUG`, `INFO`, `WARNING`, `ERROR`) | `INFO` | No |

## Updates

Home Assistant checks the repository added above periodically and shows an **Update** button on the add-on page whenever a new version is published.

## Troubleshooting

**The add-on doesn't start after an update (shows as "Stopped")**

If PP Reader has been working fine for a while and suddenly won't start after you install an update, check the `portfolio_path` entry (labeled **Portfolio file path**) in the add-on's **Configuration** tab. Open it, check the value, and save it again — if it's no longer valid, Home Assistant now tells you so directly. A valid path always starts with `/share/`, `/config/`, or `/media/` and ends in `.portfolio` (for example `/share/portfolios/my_portfolio.portfolio`).

You won't see an error about this in the PP Reader app itself — only in Home Assistant's own Supervisor system log. You don't need to read that log: re-checking and re-saving `portfolio_path` as described above is enough to fix it.

## Changelog

See [CHANGELOG.md](https://github.com/Freakandi/ha-apps/blob/main/pp-reader/CHANGELOG.md) for the full version history.

## License

MIT — see the [MIT License](https://opensource.org/license/MIT) text for details.
