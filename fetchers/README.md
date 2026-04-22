# fetchers/ — per-instance fetcher registrations

`registrations.yaml` is the **authoritative** source of truth for which
fetcher landers (SEC, ClinicalTrials, Marketaux, NewsAPI, Yahoo RSS, …)
serve which domains in this instance.

As of Wave A (2026-04-22), `kgspin_demo_app.domain_fetchers` loads this
file lazily at first access (via the `KGSPIN_DEMO_CONFIG_PATH` env var,
exported by `scripts/start-demo.sh`) and the
`kgspin-demo-register-fetchers` CLI iterates over it to write FETCHER
records into admin. There is no Python-side dict to keep in sync — edit
this YAML and re-run the CLI.

## Format

```yaml
domains:
  <domain_id>:
    fetchers:
      - <fetcher_id>
      - <fetcher_id>
  <domain_id>:
    fetchers:
      - <fetcher_id>
```

- **`<domain_id>`** — short lowercase label for the domain
  (`financial`, `clinical`, `energy`, …). Must match the domain codes
  your pipelines + bundles address.
- **`<fetcher_id>`** — the backend-named lander ID, matching the
  lander's `name` attribute and admin's FETCHER record id. See
  `kgspin-demo-app` ADR-004 on backend-named landers. A fetcher ID may
  appear under **multiple** domains (e.g. a domain-agnostic news API).

All fetcher IDs referenced here must exist in
`kgspin_demo_app.cli.register_fetchers._LANDER_CATALOG`, which maps each
ID to the Python class that implements it. Adding a new fetcher ID here
without a catalog entry will fail registration with a clear error.

## Operator workflow

Forking this repo to stand up your own instance:

1. Edit the `domains:` mapping to list your domains and which fetcher
   IDs serve each.
2. Ensure every ID you list is present in kgspin-demo-app's
   `_LANDER_CATALOG` (or add a lander + catalog entry in kgspin-demo-app
   first).
3. Re-run `uv run kgspin-demo-register-fetchers` from kgspin-demo-app.
   The CLI is idempotent — re-registering an existing FETCHER is a
   no-op (409 → return existing).

## What NOT to put here

- **Lander implementation code.** Landers live in `kgspin-demo-app`.
- **Per-domain metadata beyond the ID list.** This file is *only* the
  domain → fetcher_id mapping; lander descriptions, versions, and
  contract versions come from the lander classes themselves via
  `_LANDER_CATALOG` introspection.
- **Credentials / API keys.** Lander auth is env-var driven
  (`MARKETAUX_API_KEY`, `NEWSAPI_API_KEY`, …). Never commit secrets.

## Related

- `kgspin-demo-app/src/kgspin_demo_app/domain_fetchers.py` — the YAML
  loader + `DOMAIN_FETCHERS` mapping facade.
- `kgspin-demo-app/src/kgspin_demo_app/cli/register_fetchers.py` — the
  CLI that writes FETCHER records to admin.
- `kgspin-demo-app/docs/architecture/decisions/ADR-004-backend-named-landers.md`
  — rationale for backend-named fetcher IDs.
