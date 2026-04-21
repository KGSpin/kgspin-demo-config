# bundles/ — customer bundle overrides

Drop bundle YAMLs here to override blueprint bundles **by id**. Blueprint bundles (shipped in `kgspin-blueprint/references/bundles/`) are the default; a bundle listed here with the same id replaces the upstream version for this instance.

Leave empty unless you need to diverge from the blueprint. The kgspin-demo instance does not currently override any blueprint bundles.

## When to override

- Pinning a stricter / looser model set for a domain.
- Compliance-required extractor configurations.
- Testing a bundle variant without publishing upstream.

## File layout

One YAML per bundle, named by bundle id (e.g. `financial-v4.1.0-structural.yaml`). Schema matches the upstream blueprint bundle format — see `kgspin-blueprint/references/bundles/`.
