# Template distribution

Template source code lives in a git repository. A single repository can host a **collection** of Templates.

## Repository layout

```
my-templates/
├── README.md
└── src/
    ├── node-postgres/
    │   ├── devcontainer-template.json
    │   └── .devcontainer/
    │       ├── devcontainer.json
    │       ├── Dockerfile
    │       └── docker-compose.yml
    └── go-minimal/
        ├── devcontainer-template.json
        └── .devcontainer.json
```

Rules:
- Each Template directory under `src/` must be named exactly the `id` from its `devcontainer-template.json`.
- Each Template contains `devcontainer-template.json` plus a devcontainer config (`.devcontainer.json` or `.devcontainer/devcontainer.json`).
- Templates and Features must live in **separate repositories**.
- The Template **namespace must differ from the Feature collection namespace** when published. E.g. `ghcr.io/owner/features/...` and `ghcr.io/owner/templates/...`.

## Versioning

- Each Template is versioned independently with semver.
- Bump `version` in `devcontainer-template.json` on every behavior change.

## Packaging

Each Template is packaged as a tarball named `devcontainer-template-<id>.tgz` containing the contents of `src/<id>/`.

## OCI distribution

```
<registry>/<namespace>/<id>[:version]
```

Examples:
- `ghcr.io/owner/templates/node-postgres:1.2.0`
- `ghcr.io/owner/templates/node-postgres:1`

## CLI surface

- `devcontainer templates publish` — package and push the collection.
- `devcontainer templates apply --template-id <oci-ref> --template-args '{...}'` — scaffold a Template into a target directory.

See `publishing-workflows.md` for full command examples and CI integration.

## Discovery

Once published, Templates can be discovered by tools that index OCI registries. Supporting clients (VS Code, GitHub Codespaces, the Dev Container CLI) read the collection metadata and present Templates with their `name`, `description`, `keywords`, `platforms`, and `options`.
