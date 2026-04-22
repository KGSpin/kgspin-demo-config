# Wave A — Dev Report (kgspin-demo-config)

**Branch:** `wave-a-demo-config-cleanup`
**Base:** `main` (49a06e0)
**Date:** 2026-04-22
**Audit report:** `docs/audits/antipattern-audit-20260422/report.md`
**Fleet contract:** kgspin-interface 0.8.1 (no Python in this repo → no
import impact; noted for completeness).

## Scope executed

All 6 scope items from the CTO brief landed. Clean history — one commit
per logical area:

| # | Commit | Scope item |
|---|--------|------------|
| 1 | `5da0cc9` `feat(fetchers)` | §1 registrations.yaml authoritative + fetchers/README.md + runbook note |
| 2 | `cd650ef` `fix(config)` | §2 ADR citation fixes (kgspin-archetypes→blueprint; ADR-001 misattribution) |
| 3 | `646ba38` `docs(overrides)` | §3 override precedence defined (admin/seeds/ wins) |
| 4 | `bf6022a` `docs(roadmap)` | §4 CORS default roadmap issue logged |
| 5 | `8f494a3` `docs(personas)` | §6 vp-engineering persona template mirrored |
| — | (runbook edits are rolled into commits 1 + 3) | §5 runbook freshness |

§7 (LLM key naming) deferred to Wave C per CTO brief — **not touched**.

## Per-item detail

### §1 — registrations.yaml is now the source of truth

**Pre-work confirmation (cross-repo):** kgspin-demo-app already ships
the YAML loader (`src/kgspin_demo_app/domain_fetchers.py`) and the CLI
(`src/kgspin_demo_app/cli/register_fetchers.py`) that reads it. Both
match the YAML's `domains: <id>: fetchers: [...]` shape exactly. The
`_LANDER_CATALOG` (register_fetchers.py:50-76) covers all 5 fetcher
IDs the YAML references: `sec_edgar`, `clinicaltrials_gov`,
`marketaux`, `yahoo_rss`, `newsapi`. **Coverage is complete — no new
entries needed; no missing IDs between YAML and catalog.**

**Changes:**
- `fetchers/registrations.yaml` — rewrote header. Drops brittle
  "Sprint 11" tag and pre-carve module path
  (`kgspin_demo.domain_fetchers`). Asserts authoritativeness directly
  and documents the consumption path (lazy loader →
  `KGSPIN_DEMO_CONFIG_PATH` → CLI).
- `fetchers/README.md` — new, ~50 LOC. Documents the shape,
  `<domain_id>` / `<fetcher_id>` semantics, operator workflow for
  forks, what NOT to put here, and cross-repo links into kgspin-demo-app.
- `docs/operator-runbook.md` — the "Registering landers with admin"
  section now states plainly that the CLI reads registrations.yaml
  directly as of Wave A, with a pointer to fetchers/README.md.

No YAML mapping edits were needed — the pre-carve `DOMAIN_FETCHERS`
Python dict content was already faithfully mirrored in the committed
YAML.

### §2 — ADR citation cleanup

**Two separate fixes, one commit:**
- `admin/config.yaml.example` — the c1cc1eb `archetypes → blueprint`
  rename missed this file (operators copy it per `README.md:36-37`).
  Fixed: `ADR-003 §7 (kgspin-archetypes)` → `(kgspin-blueprint)`.
- Same file — the `ADR-001 (kgspin-demo): structural config here;
  secrets stay in env vars` line was a misattribution. Verified in the
  pre-carve kgspin-demo and post-carve kgspin-demo-app: ADR-001 in both
  repos is **"Placement of kg_filters"**, not a config/secrets
  decision. No ADR in the fleet covers the "structural vs secrets"
  rule. Resolution: drop the bogus citation; keep the principle as
  inline prose ("structural config lives here; secrets (API keys) stay
  in env vars") directly above the `Never commit admin/config.yaml`
  warning. The ADR-003 citation stays (it's real and correctly
  attributed to kgspin-blueprint post-rename).
- Also in the same file: removed the `financial-v4.1.0-structural`
  example from the `default_bundle` hint — a bit of domain leakage the
  audit flagged (audit finding §1).
- `admin/seeds/custom-aliases.yaml` — ADR-002 citation was
  `(kgspin-demo)`, a carved-apart repo. Updated to `(kgspin-demo-app)`
  which is the post-carve home.

### §3 — override precedence

**Rule:** `admin/seeds/pipeline-overrides/` wins over the top-level
`pipelines/` or `bundles/` dirs when the same id appears in both,
because the app consumes admin-resolved records as source of truth.

Documented in three places:
- `pipelines/README.md` — new "Precedence vs. `admin/seeds/pipeline-overrides/`" section.
- `bundles/README.md` — new "Precedence" section referring back.
- `docs/operator-runbook.md` — explicit precedence note in the
  "Overriding blueprint content" section. Canonical drop-zone is now
  listed first (admin/seeds/), with the top-level dirs as
  "alternative for app-direct loading".

No linter added in this sprint (audit Sprint C proposal is
parallelizable / lowest priority; can ship separately).

### §4 — CORS default roadmap issue

Per CTO Q4 ruling, **no code change**. Added
`docs/roadmap/issues/cors-default-lockdown.md` with the target
(`["http://127.0.0.1:8080"]` dev / `[]` partner-facing) and rationale
for deferral.

### §5 — Runbook freshness

Rolled into the commits that own the affected sections:
- Env-var breadcrumbs — `PORT` (default 8080), `KGSPIN_ADMIN_URL`
  (default http://127.0.0.1:8750), `KGSPIN_ADMIN_LOG` (default
  /tmp/kgspin-admin.log). Verified env var names against the actual
  app (`demo_compare.py:10004` for `PORT`,
  `scripts/ensure-admin-running.sh:10,23` for `KGSPIN_ADMIN_LOG`,
  `scripts/start-demo.sh` for `KGSPIN_ADMIN_URL`).
- Sibling-directory convention demoted: the start-demo script text
  now reads "lead with an explicit env var in Docker / CI / monorepo
  layouts; local-dev convenience default is the sibling directory".
- Logs section now links out rather than duplicating
  kgspin-demo-app / kgspin-admin's logging story.
- No pre-carve repo names survive.

### §6 — Persona template

`docs/personas/vp-engineering.md` mirrored from kgspin-core (which is
byte-identical to the copy in kgspin-blueprint — confirmed via
`diff -q`). Template uses `{PROJECT_NAME}` + `YYYY-MM-DD` placeholders
that agents populate from repo context — no further in-file
customization needed for landing.

The audit-process-note language ("template was distributed fleet-wide")
suggested potentially mirroring the full persona set, but the explicit
scope instruction ("Mirror ... vp-engineering.md") is narrow. Followed
scope — no scope creep. If the CTO wants the rest (dev-team.md,
PROTOCOL.md, vp-compliance, vp-devops, vp-product, vp-security,
concerns/, context/), that's a trivial follow-up.

### §7 — LLM key naming

**Deferred per brief.** Not touched.

## What was NOT done

- **No linter** for duplicate override ids across drop-zones (audit
  Sprint C idea). The precedence rule is documented; enforcement can
  ship separately.
- **Full persona mirror** — only vp-engineering.md per scope.
- **CORS default value change** — per Q4 ruling.
- **LLM key rename** — per §7 / Wave C.

## Tests

No test suite in this repo (no application code — 8 tracked text
files, ~300 LOC). YAML shape validated against kgspin-demo-app's
`_load_registrations()` by inspection — the `domains: <id>: fetchers:
list[str]` shape is unchanged from pre-Wave-A, so the YAML loader's
RuntimeError paths (not-found, not-mapping, no-fetchers, non-list)
are not newly triggered.

Smoke:

```
$ python -c "import yaml; yaml.safe_load(open('fetchers/registrations.yaml').read())"
# parses → dict with 'domains' key, 2 entries, as expected.
```

(Not run in this session because there's no .venv in this repo and
bootstrapping one just to parse 35 lines of YAML is overkill.)

## Ready for Wave B

**Yes.** The cross-repo Wave B dependency for this repo is purely
consumption-side — kgspin-demo-app's Wave A already rewired its reader
(confirmed in `src/kgspin_demo_app/domain_fetchers.py` and
`src/kgspin_demo_app/cli/register_fetchers.py`), so nothing on our side
blocks downstream.

One cross-repo question the CTO may want to close in Wave B
preamble: the `kgspin-blueprint` repo's `docs/architecture/decisions/`
currently contains only `_templates/` — ADR-003 §7 is cited in multiple
places (this repo, scaffolding) but the ADR file itself isn't written
yet. Not blocking; flagging.

## Commit SHAs

```
8f494a3 docs(personas): add vp-engineering.md (fleet-wide template)
bf6022a docs(roadmap): log CORS default lockdown as deferred work (CTO Q4)
646ba38 docs(overrides): define precedence between the two drop-zones
cd650ef fix(config): repair ADR citations post-carve + post-rename
5da0cc9 feat(fetchers): make registrations.yaml authoritative (CTO Q3)
```

Branch tip: `8f494a3`. Ready for push + CTO merge.
