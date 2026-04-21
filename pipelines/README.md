# pipelines/ — customer pipeline overrides

Drop pipeline YAMLs here to override blueprint pipelines **by id**. Blueprint pipelines (shipped in `kgspin-blueprint/references/pipelines/`) are the default; a pipeline listed here with the same id replaces the upstream version for this instance.

Leave empty unless you need to diverge from the blueprint. The kgspin-demo instance does not currently override any blueprint pipelines.

## When to override

- Custom preprocessor chain for a regulated domain.
- Experimental LLM routing not yet upstreamed.
- Per-customer prompt or rubric tweaks.

## File layout

One YAML per pipeline, named by pipeline id. Schema matches the upstream blueprint pipeline format — see `kgspin-blueprint/references/pipelines/`. The `admin/seeds/pipeline-overrides/` directory is an alternative drop-zone consumed by `kgspin-admin import-pipelines`; pick whichever flow suits your deployment.
