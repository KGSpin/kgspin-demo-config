# Roadmap issue — lock down CORS default

**Status:** deferred (CTO Q4, Wave A 2026-04-22) — logged, not fixed.
**Owner:** TBD (pre-first-partner engagement).

## The issue

`admin/config.yaml.example:29-30` ships `cors_origins: ["*"]` as the
default. A comment warns "dev default", but operators who copy-paste
the file verbatim will inherit a wide-open CORS policy on the FastAPI
`/extract/*` endpoints.

## Target

- Dev default: `["http://127.0.0.1:8080"]` (matches the demo UI's own
  origin).
- Partner-facing / shipping deployments: `[]` (empty — explicit
  allowlist required before the API answers any browser origin).
- Move `"*"` into a commented-out line as the "I know what I'm doing"
  escape hatch rather than the default.

## Why not fix now

CTO Q4 ruling: Wave A scope is rename + citation + documentation
cleanup. Changing a shipped default is a behavioral change that risks
breaking any operator who's quietly been relying on the wide-open
default in a trusted network. Track here; revisit before the first
design-partner ships.
