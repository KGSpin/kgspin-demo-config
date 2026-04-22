# bundles/ — customer bundle overrides

Drop bundle YAMLs here to override blueprint bundles **by id**. Blueprint bundles (shipped in `kgspin-blueprint/references/bundles/`) are the default; a bundle listed here with the same id replaces the upstream version for this instance.

Leave empty unless you need to diverge from the blueprint. The kgspin-demo instance does not currently override any blueprint bundles.

## When to override

- Pinning a stricter / looser model set for a domain.
- Compliance-required extractor configurations.
- Testing a bundle variant without publishing upstream.

## File layout

One YAML per bundle, named by bundle id. Schema matches the upstream blueprint bundle format — see `kgspin-blueprint/references/bundles/`.

## Precedence

Bundles follow the same precedence rule as pipeline overrides: if a
bundle id is defined in both `admin/seeds/` (admin-imported) and in
this `bundles/` drop-zone, the **admin-imported bundle wins** because
the app reads admin-resolved bundles as the source of truth. Keep each
bundle id in exactly one dir.
