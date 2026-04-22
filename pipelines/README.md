# pipelines/ — customer pipeline overrides

Drop pipeline YAMLs here to override blueprint pipelines **by id**. Blueprint pipelines (shipped in `kgspin-blueprint/references/pipelines/`) are the default; a pipeline listed here with the same id replaces the upstream version for this instance.

Leave empty unless you need to diverge from the blueprint. The kgspin-demo instance does not currently override any blueprint pipelines.

## When to override

- Custom preprocessor chain for a regulated domain.
- Experimental LLM routing not yet upstreamed.
- Per-customer prompt or rubric tweaks.

## File layout

One YAML per pipeline, named by pipeline id. Schema matches the upstream blueprint pipeline format — see `kgspin-blueprint/references/pipelines/`.

## Precedence vs. `admin/seeds/pipeline-overrides/`

There are two drop-zones for pipeline overrides in this repo:

1. **`admin/seeds/pipeline-overrides/` (canonical)** — consumed by
   `kgspin-admin import-pipelines`. This is admin's working directory
   and is the intended home for overrides you want admin to import on
   each boot. **Use this one by default.**
2. **`pipelines/` (this directory)** — a second drop-zone for overrides
   loaded by the demo app's pipeline resolver directly. Useful when you
   want to diverge from blueprint without going through `kgspin-admin
   import-pipelines`.

**If the same pipeline id appears in both dirs, `admin/seeds/pipeline-overrides/`
wins** (admin's import runs last; admin-resolved pipelines are the
source of truth the app reads). Don't rely on this — pick one dir per
pipeline id and keep the other empty for that id.
