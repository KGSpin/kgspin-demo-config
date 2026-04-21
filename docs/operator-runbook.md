# Operator runbook — kgspin-demo instance

This runbook describes how the **kgspin-demo** instance is deployed. Fork it for your own instance.

## Prerequisites

- `uv` installed (Python package manager).
- Python 3.11+.
- Sibling clones of:
  - `kgspin-archetypes` (upstream blueprint)
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
1. Resolves `KGSPIN_DEMO_CONFIG_PATH` (default: sibling `../kgspin-demo-config`).
2. Exports `KGSPIN_DEMO_CONFIG=$KGSPIN_DEMO_CONFIG_PATH/admin/config.yaml` if the file exists.
3. Starts `kgspin-admin` in the background if not already responding.
4. Launches the FastAPI compare UI on port 8080.

Open <http://127.0.0.1:8080/intelligence.html> to reach the Intelligence tab.

## Overriding blueprint content

If you need pipeline or bundle variants that differ from the upstream blueprint:

1. Drop the YAML into `pipelines/<pipeline-id>.yaml` or `bundles/<bundle-id>.yaml`.
2. Or drop it into `admin/seeds/pipeline-overrides/` and re-run `kgspin-admin import-pipelines`.
3. Re-start the demo — admin reloads the registry on boot.

The blueprint remains authoritative for anything **not** overridden here.

## Registering landers with admin

```bash
cd ~/repos/kgspin-demo-app
uv run kgspin-demo-register-fetchers
```

This reads `kgspin-demo-config/fetchers/registrations.yaml` (via the app's fetcher registry) and writes records to admin.

## Logs

- Admin: `/tmp/kgspin-admin.log`
- Demo compare UI: stdout of `start-demo.sh`
- Landers: stdout when invoked directly; per-lander files under `$corpus_root/logs/` for batch runs.
