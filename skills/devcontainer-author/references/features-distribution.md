# Feature distribution

Feature source code lives in a git repository. A single repository can host a **collection** of Features.

## Repository layout

```
my-features/
├── README.md
├── src/
│   ├── feature-a/
│   │   ├── devcontainer-feature.json
│   │   └── install.sh
│   └── feature-b/
│       ├── devcontainer-feature.json
│       └── install.sh
└── test/
    ├── feature-a/
    │   └── test.sh
    └── feature-b/
        └── test.sh
```

Rules:
- Each Feature directory under `src/` must be named exactly the `id` from its `devcontainer-feature.json`.
- One `devcontainer-feature.json` per Feature directory.
- Tests under `test/<feature-id>/` mirror `src/`.
- Do not mix Features and Templates in the same repository — Templates belong in a separate repo.

## Versioning

- Each Feature is versioned independently with semver.
- Bump `version` in `devcontainer-feature.json` on every behavior change.
- Tag releases per Feature when publishing manually; CI-driven publishing reads the `version` field.

## Packaging

Each Feature is packaged as a tarball named `devcontainer-feature-<id>.tgz` containing the contents of `src/<id>/`.

## OCI distribution

Features are published as OCI artifacts. The reference form is:

```
<registry>/<namespace>/<id>[:version]
```

Examples:
- `ghcr.io/owner/features/my-tool:1.0.0`
- `ghcr.io/owner/features/my-tool:1`
- `ghcr.io/owner/features/my-tool` (resolves to `latest`)

Semver tags are pushed at multiple precisions (`1`, `1.0`, `1.0.0`) so consumers can pin loosely.

## Collection metadata

When publishing, a `devcontainer-collection.json` is generated. It lists the collection's Features with their metadata so registries and tools can discover them. The `devcontainer features publish` CLI command produces it automatically — do not hand-edit.

## Consumer reference

Consumers reference Features by their OCI path in `devcontainer.json`:

```json
{
  "features": {
    "ghcr.io/owner/features/my-tool:1": { "version": "latest" }
  }
}
```

See `publishing-workflows.md` for the CLI commands and GitHub Actions used to publish.
