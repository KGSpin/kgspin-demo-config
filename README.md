# kgspin-demo-config

**Reference customer instance config** — Layer 2 in the KGSpin deployment model.

This repo holds the operator-managed configuration for the KGSpin demo instance. It sits *between* the upstream blueprint (`kgspin-blueprint`) and the application (`kgspin-demo-app`). Operators fork this repo to stand up their own instance without modifying the app or the blueprint.

## Layer model

| Layer | Repo | Purpose |
|-------|------|---------|
| 1 — Blueprint | `kgspin-blueprint` | Upstream-curated pipelines, bundles, LLM alias catalog |
| **2 — Config (this repo)** | `kgspin-demo-config` | Per-instance overrides + fetcher registrations + admin config |
| 3 — App | `kgspin-demo-app` | Long-running services + UI + benchmarks |

See `kgspin-blueprint` ADR-003 for the full pattern.

## What lives here

```
admin/
  config.yaml.example          # copy to config.yaml (gitignored) and edit
  seeds/
    custom-aliases.yaml        # per-instance LLM alias overrides (blueprint wins unless listed)
    pipeline-overrides/        # YAMLs that replace blueprint pipelines by id
fetchers/
  registrations.yaml           # which fetcher IDs serve which domains (instance choice)
bundles/                       # override blueprint bundles (empty by default)
pipelines/                     # override blueprint pipelines (empty by default)
docs/
  operator-runbook.md          # deployment procedure for this instance
```

## First-run

```bash
cp admin/config.yaml.example admin/config.yaml
# edit admin/config.yaml — fill paths + instance config (secrets stay in env vars)
export KGSPIN_DEMO_CONFIG_PATH="$(pwd)"
cd ../kgspin-demo-app && ./scripts/start-demo.sh
```

The app resolves this repo via `KGSPIN_DEMO_CONFIG_PATH` (default: sibling directory `../kgspin-demo-config`).

## What NOT to put here

- **No application code.** App logic lives in `kgspin-demo-app`.
- **No upstream pipeline definitions.** Those live in `kgspin-blueprint`; override here only when you must diverge.
- **No secrets in committed files.** `admin/config.yaml` is gitignored; use env vars for API keys.

## Related

- Upstream blueprint: `kgspin-blueprint`
- App repo: `kgspin-demo-app`
- Admin registry: `kgspin-admin`
