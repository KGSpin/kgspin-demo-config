# Wave B — Dev Report (kgspin-demo-config)

**Branch:** `wave-b-demo-config-polish`
**Base:** `main` (95d0ae1 — Wave A merged)
**Date:** 2026-04-22
**CTO brief:** `/tmp/cto/wave-b-20260422/kgspin-demo-config.md`
**Fleet contract:** kgspin-interface 0.8.1 (still no Python in this
repo → no import impact; noted for completeness).

## Scope executed

Light-touch polish pass as scoped by CTO. All four brief items landed.
Clean history — one commit per logical area:

| # | Commit  | Scope item |
|---|---------|------------|
| 1 | `2c71976` `docs(runbook)`  | §2 Runbook walk-through (new-customer read, fixed every lie/dangling ref) |
| 2 | `ce00d9f` `docs(fetchers)` | §3 YAML format documentation (per-source example blocks) |
| 3 | `3d57d27` `docs(personas)` | §4 Persona template sync (full fleet mirror) |

§1 (Wave A spillover) required no new code — see §1 below.

## Per-item detail

### §1 — Wave A spillover

**Nothing actionable at the Wave B level.** Wave A's dev-report
(`docs/sprints/wave-a-demo-config-cleanup/dev-report.md`) listed four
deferred items, and the Wave B brief's resolution for each stands:

- **No linter for duplicate override ids** across drop-zones — audit
  Sprint C idea; still parallelizable, still not blocking. No action.
- **Full persona mirror** — addressed as a Wave B scope item (§4); see
  below.
- **CORS default value change** — CTO Q4 ruling was to document under
  `docs/roadmap/issues/cors-default-lockdown.md`, not change code.
  Still correct.
- **LLM key naming (§7)** — deferred to Wave C per original brief.
  Still untouched.

Cross-repo flag from the Wave A dev-report (the ADR-003 §7 citation
refers to a file that lives only as `_templates/` in kgspin-blueprint)
is a kgspin-blueprint concern, not something this repo can land.

### §2 — Runbook walk-through

Read `docs/operator-runbook.md` top-to-bottom as a new customer would,
then cross-checked every concrete claim against the sibling repos.
Four factual errors found and fixed:

1. **`/intelligence.html` does not exist.** The FastAPI app's only
   HTML route is `/` (serves `compare.html`). The Intelligence tab is
   rendered inside that page.
   - Evidence: `kgspin-demo-app/demos/extraction/demo_compare.py:1181-1187`
     (the `/` handler is the only HTML route); no `intelligence.html`
     anywhere in `demos/extraction/static/`.
   - Fix: changed the "Open …" line to point at `/` with a note
     explaining that Intelligence is a tab, not a page.

2. **`NEWSAPI_API_KEY` is the wrong var name.** The newsapi lander
   reads `NEWSAPI_KEY`.
   - Evidence: `kgspin-demo-app/src/kgspin_demo_app/landers/newsapi.py:115`
     (`os.environ.get("NEWSAPI_KEY", "")`) and `newsapi.py:268` (its
     declared env-var name).
   - Fix: renamed in the runbook's secrets block.

3. **`EDGAR_IDENTITY` was missing entirely** — required for
   `sec_edgar` per SEC compliance. The lander raises `FetcherError`
   at call time if neither `EDGAR_IDENTITY` nor the legacy
   `SEC_USER_AGENT` fallback is set.
   - Evidence: `kgspin-demo-app/src/kgspin_demo_app/landers/sec.py:182-193`.
   - Fix: added to the secrets block with the expected format
     ("Your Name your@email.com").

4. **"admin reloads the registry on boot" is wrong.** Admin's
   auto-sync is opt-in; default is off.
   - Evidence: `kgspin-admin/src/kgspin_admin/serve.py:132-144`
     (`_maybe_auto_sync_blueprint` returns early unless
     `features.auto_sync_blueprint` is true) and `serve.py:239-243`
     (the commentary explaining that registrations are explicit
     operator actions, not hidden bootstrap magic). Also:
     `kgspin-admin/README.md:205` documents the default as unset/off.
   - Fix: rewrote the "Overriding blueprint content" section to say
     re-import is an explicit step (`kgspin-admin import-pipelines` /
     `import-aliases`), with the opt-in
     `KGSPIN_ADMIN_AUTO_SYNC_BLUEPRINT=1` noted for dev convenience.
     Also called out that auto-sync only covers blueprint pipelines +
     aliases, not fetcher registrations (those are always explicit
     via `kgspin-demo-register-fetchers`).

Also split the API-key block per lander with required/optional
annotations matching the set in `registrations.yaml`:
`EDGAR_IDENTITY` (sec_edgar, required), `MARKETAUX_API_KEY`
(marketaux, required), `NEWSAPI_KEY` (newsapi, required),
`CLINICAL_TRIALS_API_KEY` (clinicaltrials_gov, optional), none
(yahoo_rss).

**Things I verified but did not change** (runbook was correct on
these):

- `./scripts/start-demo.sh` path — EXISTS at
  `kgspin-demo-app/scripts/start-demo.sh`.
- `KGSPIN_DEMO_CONFIG` export behaviour — matches the actual script
  lines 37-39.
- Admin defaults `http://127.0.0.1:8750` and `/tmp/kgspin-admin.log`
  — match `ensure-admin-running.sh:19,23`.
- `PORT` default `8080` — matches `demo_compare.py:9640`.
- `uv run kgspin-demo-register-fetchers` CLI name — matches
  `kgspin-demo-app/pyproject.toml:66`.
- `kgspin-admin import-pipelines` command — registered in
  `kgspin-admin/src/kgspin_admin/registry/import_pipelines_cli.py:39-42`.
- `kgspin-core` / `kgspin-interface` sibling clones required — yes,
  both are declared as editable path deps in
  `kgspin-demo-app/pyproject.toml:70-71`.

### §3 — YAML format documentation

Verified `fetchers/README.md` against the actual YAML loader
(`kgspin-demo-app/src/kgspin_demo_app/domain_fetchers.py`):

- Loader reads only `domains.<domain_id>.fetchers` (list of strings).
- Error paths: missing `domains` key; `domains` not a mapping; domain
  entry missing `fetchers`; `fetchers` not a list / non-string items.
- **No optional fields.** Any other keys are silently ignored — no
  description, env, priority, version, contract hints are parsed.
- The `_LANDER_CATALOG` ships 5 IDs: `sec_edgar`, `clinicaltrials_gov`,
  `marketaux`, `yahoo_rss`, `newsapi`. All 5 are referenced by the
  committed `registrations.yaml` (same finding as Wave A).

The existing schema section in the README was accurate; what it
lacked was a per-source worked example. Added:

- A **catalog table** listing all 5 shipped lander IDs, what each
  fetches, the auth env var (or "none"), and which domain this repo
  ships it under.
- **One copy-pasteable YAML block per lander** showing it registered
  under an appropriate domain — matches the wording in the CTO brief
  ("example YAML entry per supported source").
- **A combined block** matching `registrations.yaml` verbatim, so
  operators forking the repo can see the full shape at a glance.
- A closing note that adding a fetcher beyond this catalog is a
  kgspin-demo-app change (implement lander + add to `_LANDER_CATALOG`),
  not a YAML-only edit — echoes the existing "What NOT to put here"
  guidance.

No change to `registrations.yaml` itself (shape is already correct).

### §4 — Persona template sync

**`vp-engineering.md` is byte-identical** to the copies in kgspin-core
and kgspin-blueprint (verified by md5; all three hash
`f94d588b7a911fffad36b7abbc23dc20`). No drift since Wave A landed the
file.

**Other persona files were missing.** Inventory across siblings:

| File | blueprint | core | admin | interface | demo-config (pre) |
|------|:---:|:---:|:---:|:---:|:---:|
| `dev-team.md` | ✓ | ✓ | ✓ | ✓ | ✗ |
| `PROTOCOL.md` | ✓ | ✓ | ✓ | ✓ | ✗ |
| `vp-product.md` | ✓ | ✓ | ✓ | ✓ | ✗ |
| `vp-security.md` | ✓ | ✓ | ✓ | ✓ | ✗ |
| `vp-compliance.md` | ✓ | ✓ | ✓ | ✓ | ✗ |
| `vp-devops.md` | ✓ | ✓ | ✓ | ✓ | ✗ |
| `vp-engineering.md` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `concerns/compliance.md` | ✓ | ✓ | ✓ | ✓ | ✗ |
| `concerns/devops.md` | ✓ | ✓ | ✓ | ✓ | ✗ |
| `concerns/security.md` | ✓ | ✓ | ✓ | ✓ | ✗ |
| `context/README.md` | ✓ | ✓ | ✓ | ✓ | ✗ |

Wave A intentionally narrowed to `vp-engineering.md` per scope. Wave B
brief asks to "Confirm the other persona files … are also present or
flag as missing." All 10 were missing in this repo but present in
every other fleet repo. Two options: flag as missing and punt, or
mirror them now. Mirrored them — the Wave A dev-report already
described this as "a trivial follow-up," the source files are
byte-identical across siblings, and the brief's precedent (Wave A
vp-engineering.md) was to mirror rather than flag.

Source: copied from kgspin-core (byte-identical to kgspin-blueprint).
Placeholders `{PROJECT_NAME}` and `YYYY-MM-DD` stay as-is — populated
by agents from repo context at invocation time.

**No CTO persona file was added.** None of the sibling repos carry
one (CTO operates fleet-wide), so parity = no cto.md here. If that's
wrong, happy to add; I followed the fleet shape.

## What was NOT done

- **No scope creep beyond the four brief items.** No changes to
  `registrations.yaml`, `admin/config.yaml.example`, `bundles/README.md`,
  `pipelines/README.md`, or the override precedence rule — Wave A
  settled all of those.
- **No CORS default change** — still deferred per roadmap issue.
- **No LLM key rename** — still scheduled for Wave C.
- **No linter for duplicate override ids.**

## Tests

Still no test suite in this repo (documentation-only changes, still
~300 LOC of text across ~18 files now with the persona mirror).

Smoke:

```
$ python -c "import yaml; yaml.safe_load(open('fetchers/registrations.yaml').read())"
# parses → dict with 'domains' key, 2 entries, as expected (unchanged from Wave A).
```

(Not run in this session — same reasoning as Wave A: no .venv, no
value in bootstrapping one to re-parse 35 unchanged lines of YAML.)

Runbook claims were verified by direct file reads in sibling repos;
see §2 above for the evidence chain per claim.

## Ready for Wave C

**Yes.** Nothing in Wave B introduces downstream dependencies. The
Wave C LLM-key rename remains the outstanding Layer-2 item — the
configs that reference `llm.default_alias` / `llm.compare_qa_llm` /
etc. in `admin/config.yaml.example` will need mechanical renaming
once Wave C lands the app-side change.

## Commit SHAs

```
3d57d27 docs(personas): mirror remaining fleet persona templates
ce00d9f docs(fetchers): add per-source YAML examples matching loader schema
2c71976 docs(runbook): fix dangling refs found on new-customer walk-through
```

Branch tip: `3d57d27`. Ready for push + CTO merge.
