# Feature authoring

A Feature is a reusable, installable unit (CLI tool, runtime, package manager, organization tooling) that augments a primary dev container. Features are referenced from `devcontainer.json`:

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/node:1": { "version": "lts" },
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  }
}
```

Feature option keys and values vary per Feature. Inspect the Feature's `devcontainer-feature.json` before inventing options.

## When to use a Feature

Good candidates:
- CLI tools (`gh`, `kubectl`, `terraform`)
- Language runtimes (Node, Python, Go, Java)
- Package managers (pnpm, poetry, uv)
- Shared organization tooling
- Common utilities (shell completions, build tools)

Bad candidates (put these in the repo, Dockerfile, or lifecycle commands instead):
- Project-specific application dependencies
- Secrets
- Local developer preferences
- One-off repository bootstrap (running migrations, seeding DBs)

## Authoring a single Feature

Minimum two files in `src/<feature-id>/`:

### `devcontainer-feature.json`

```json
{
  "id": "my-tool",
  "version": "1.0.0",
  "name": "My Tool",
  "description": "Installs my-tool CLI.",
  "documentationURL": "https://example.com/my-tool",
  "options": {
    "version": {
      "type": "string",
      "proposals": ["latest", "1.2.3"],
      "default": "latest",
      "description": "Version of my-tool to install."
    },
    "enableShellCompletions": {
      "type": "boolean",
      "default": true,
      "description": "Install shell completions."
    }
  },
  "installsAfter": [
    "ghcr.io/devcontainers/features/common-utils"
  ],
  "containerEnv": {
    "MY_TOOL_HOME": "/usr/local/share/my-tool"
  }
}
```

Key fields:
- `id` — must match the directory name.
- `version` — semver; bump on every change.
- `options` — typed inputs. Use `enum` for strict values, `proposals` for hints.
- `installsAfter` — Features that must run before this one (soft ordering).
- `containerEnv` / `remoteEnv` — environment exposed by the Feature.
- `mounts`, `init`, `privileged`, `capAdd`, `securityOpt`, `entrypoint` — request container modifications. Use sparingly.

### `install.sh`

```sh
#!/usr/bin/env bash
set -euo pipefail

VERSION="${VERSION:-latest}"
ENABLESHELLCOMPLETIONS="${ENABLESHELLCOMPLETIONS:-true}"

if [ "$(id -u)" -ne 0 ]; then
  echo "ERROR: install.sh must run as root."
  exit 1
fi

# Detect distro, install dependencies, fetch binary, etc.
```

Rules:
- Options are exposed as ALL-CAPS env vars matching the option key.
- Script runs as root during image build.
- Do not assume a single base image. Detect distro (`/etc/os-release`) and branch.
- Be idempotent. Features may install on top of layered images.
- Clean up apt/dnf caches at the end.

## Install order

`installsAfter` declares soft ordering between Features. When a config needs strict ordering, use `overrideFeatureInstallOrder` in `devcontainer.json`:

```json
{
  "features": {
    "ghcr.io/owner/coll/a:1": {},
    "ghcr.io/owner/coll/b:1": {}
  },
  "overrideFeatureInstallOrder": [
    "ghcr.io/owner/coll/a",
    "ghcr.io/owner/coll/b"
  ]
}
```

## Tests

Optional but recommended for non-trivial Features. Layout: `test/<feature-id>/test.sh`. Tests run inside a container with the Feature installed; use `check` from the dev container test harness (see `publishing-workflows.md`).
