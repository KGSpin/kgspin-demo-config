# Wave C — Dev Report (kgspin-demo-config)

**Branch:** `wave-c-polish`
**Base:** `main` (bb0edd7 — Wave B merged)
**Date:** 2026-04-22

## Scope executed

Light-touch polish sweep per the CTO brief. This repo is small
(documentation-only, ~18 tracked text files) and Waves A + B already
landed the heavy renames and factual fixes; the Wave C pass turned up
three carry-over items:

- `fetchers/README.md` — `NEWSAPI_API_KEY` → `NEWSAPI_KEY` (Wave B
  fixed this in `docs/operator-runbook.md:48` but missed the same
  string in the fetchers README's "What NOT to put here" block). The
  newsapi lander reads `NEWSAPI_KEY` per
  `kgspin-demo-app/src/kgspin_demo_app/landers/newsapi.py:115,268`.
- `README.md` first-run block — "edit admin/config.yaml — fill
  secrets + paths" rewritten to "fill paths + instance config
  (secrets stay in env vars)". The old wording contradicted the
  explicit "secrets (API keys) stay in env vars" rule in
  `admin/config.yaml.example:9-10` and the operator-runbook.md
  env-var table. No file-layout change.
- `docs/sprints/_templates/dev-report.md` — added. This repo had no
  `_templates/` dir at all; all sibling repos (kgspin-core,
  kgspin-interface, kgspin-blueprint, kgspin-admin) carry one. Mirrored
  the **neutralized** copy from `kgspin-interface` (its Wave C
  `383c8c4` already swapped "Ticker Intelligence tab" for "relevant
  UI surface" and "ghost entities like 'adamczyk'" for "no unexpected
  ghost entries"). kgspin-core and kgspin-admin still carry the
  pre-neutralized version — that's their Wave C to land, not ours.
  Following the Wave B precedent (persona files mirrored when missing
  fleet-wide), this unblocks future sprints in this repo from having a
  template to copy.

## Per-scope-item status

| # | Item | Status |
|---|------|--------|
| 1 | Test fixtures neutralized | **N/A** — no test suite (no Python) |
| 2 | Docstrings: ADR-036 / ADR-011 / pre-rename refs | **Clean** — none found in live docs |
| 3 | Comments: kgenskills / kgspin-archetypes / etc. | **Clean** — none found in live docs (Wave A already swept the live set; remaining matches are in the Wave A / Wave B dev-reports, which are historical and must not be rewritten) |
| 4 | Dev-report template refreshed | **Mirrored** neutralized copy from kgspin-interface |
| 5 | README + cross-repo path references | **Clean** apart from the "fill secrets + paths" wording fix above |
| 6 | Persona files — stale examples | **Clean** — Wave B mirrored these byte-for-byte from kgspin-core; no domain-specific literals (verified by grep for `ticker`, `AAPL`, `Intelligence tab`, `adamczyk`, `financial-v*`, `ADR-036`, `ADR-011`, `kgenskills`, `archetypes`) |

## Things intentionally not touched

- **Wave A / Wave B dev-reports.** These contain historical references
  to `kgspin-archetypes`, `NEWSAPI_API_KEY`, and `financial-v4.1.0`,
  but as audit trail describing what was fixed. Rewriting them would
  break the sprint-history record.
- **`{PROJECT_NAME}` / `YYYY-MM-DD` placeholders** in the mirrored
  persona files. Intentional — agents populate these at invocation
  time from repo context, per the template convention fleet-wide.
- **The "Intelligence tab" reference in `docs/operator-runbook.md:77-78`.**
  This is a factual description of the live app's UI (a tab inside
  `compare.html`), verified by Wave B against
  `kgspin-demo-app/demos/extraction/demo_compare.py:1181-1187`. It is
  not the stale **template-example** "Ticker Intelligence tab" the
  preamble calls out. Kept as-is.
- **The kgspin-interface-style template is mirrored verbatim.** No
  repo-specific customization — the whole point of a fleet template is
  to stay fleet-wide.

## Wave D spillover

None. Every touch was doc-only and one-line; nothing triggered the
"touching runtime code" carve-out in the brief.

## Tests

Still no test suite in this repo (documentation-only, ~20 tracked text
files now with the new template). Smoke:

```
$ python -c "import yaml; yaml.safe_load(open('fetchers/registrations.yaml').read())"
# parses → dict with 'domains' key, 2 entries, unchanged from Waves A+B.
```

Not run this session — the YAML was not touched.

## Commits

One commit per logical area:

| # | Commit         | Scope |
|---|----------------|-------|
| 1 | `docs(fetchers)` | `NEWSAPI_API_KEY` → `NEWSAPI_KEY` in fetchers/README.md |
| 2 | `docs(readme)`   | First-run comment: paths + instance config, secrets in env |
| 3 | `docs(sprints)`  | Mirror neutralized dev-report template from kgspin-interface |

Branch ready for push + CTO merge.
