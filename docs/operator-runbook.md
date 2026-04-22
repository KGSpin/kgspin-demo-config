# Operator runbook — kgspin-demo instance

This runbook describes how the **kgspin-demo** instance is deployed. Fork it for your own instance.

## Prerequisites

- `uv` installed (Python package manager).
- Python 3.11+.
- Sibling clones of:
  - `kgspin-blueprint` (upstream blueprint)
  - `kgspin-admin` (runtime registry service)
  - `kgspin-core`, `kgspin-interface` (libraries)
  - `kgspin-demo-app` (the running services + UI)
- Network access to Gemini (or your configured LLM provider) and the fetcher endpoints (SEC EDGAR, Marketaux, Yahoo RSS, ClinicalTrials.gov, NewsAPI).

## One-time setup

```bash
# 1. Clone this repo as the Layer 2 config.
cd ~/repos && git clone <this-repo-url> kgspin-demo-config

# 2. Copy the example config and fill real values.
cd kgspin-demo-config
cp admin/config.yaml.example admin/config.yaml
$EDITOR admin/config.yaml
```

Key fields to set in `admin/config.yaml`:

| Field | What it controls | Default |
|-------|------------------|---------|
| `storage.corpus_root` | Where landers write raw artifacts | `~/.kgspin/corpus` |
| `storage.bundles_dir` | Where compiled bundles live | `.bundles` |
| `security.cors_origins` | CORS allowlist for the demo API | `["*"]` (dev-only) |
| `features.default_bundle` | Pin a specific bundle id | auto-latest |
| `llm.default_alias` | Admin-registered LLM alias id | `null` (legacy fallback) |

Secrets (API keys) stay in env vars — never in `config.yaml`:

```bash
export KGSPIN_ADMIN_URL=http://127.0.0.1:8750
export KGSPIN_DEMO_CONFIG_PATH=/Users/you/repos/kgspin-demo-config
# Provider-specific:
export GEMINI_API_KEY=...
export MARKETAUX_API_KEY=...
export NEWSAPI_API_KEY=...
```

## Starting the demo

```bash
cd ~/repos/kgspin-demo-app
./scripts/start-demo.sh
```

The start script:
1. Resolves `KGSPIN_DEMO_CONFIG_PATH` (lead with an explicit env var in
   Docker / CI / monorepo layouts; local-dev convenience default is the
   sibling directory `../kgspin-demo-config`).
2. Exports `KGSPIN_DEMO_CONFIG=$KGSPIN_DEMO_CONFIG_PATH/admin/config.yaml` if the file exists.
3. Starts `kgspin-admin` in the background if not already responding
   (URL from `KGSPIN_ADMIN_URL`, default `http://127.0.0.1:8750`; log
   path from `KGSPIN_ADMIN_LOG`, default `/tmp/kgspin-admin.log`).
4. Launches the FastAPI compare UI (port from `PORT` env var, default `8080`).

Open <http://127.0.0.1:8080/intelligence.html> (substitute `$PORT` if
overridden) to reach the Intelligence tab.

## Overriding blueprint content

If you need pipeline or bundle variants that differ from the upstream blueprint:

1. Drop the YAML into `admin/seeds/pipeline-overrides/` (the **canonical**
   drop-zone) and re-run `kgspin-admin import-pipelines`.
2. Alternative for app-direct loading: drop into `pipelines/<pipeline-id>.yaml`
   or `bundles/<bundle-id>.yaml`.
3. Re-start the demo — admin reloads the registry on boot.

**Precedence:** if the same id exists in both `admin/seeds/` and the
top-level `pipelines/` or `bundles/` dirs, the admin-imported version
wins (the app reads admin's resolved view). Keep each id in one dir to
avoid surprise. See `pipelines/README.md` / `bundles/README.md` for the
full rule.

The blueprint remains authoritative for anything **not** overridden here.

## Registering landers with admin

```bash
cd ~/repos/kgspin-demo-app
uv run kgspin-demo-register-fetchers
```

As of Wave A (2026-04-22), this CLI reads
`kgspin-demo-config/fetchers/registrations.yaml` directly (resolved via
`KGSPIN_DEMO_CONFIG_PATH`) and writes one FETCHER record per unique
backend-named lander into admin. Edit the YAML, re-run the command —
registration is idempotent. See `fetchers/README.md` for the YAML
format.

## Logs

- Admin: `/tmp/kgspin-admin.log` (override via `KGSPIN_ADMIN_LOG`).
- Demo compare UI: stdout of `start-demo.sh`.
- Landers: stdout when invoked directly; per-lander files under
  `$corpus_root/logs/` for batch runs.

For structure / rotation / correlation, see the logging sections in
`kgspin-demo-app` and `kgspin-admin` — this runbook intentionally does
not duplicate their stories.
