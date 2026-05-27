# Template authoring

A Template packages source files that encode configuration for a complete development environment. Applying a Template scaffolds a project's `.devcontainer/` (and optional supporting files) so users can start coding immediately.

## Minimum contents

A Template directory must contain:
1. `devcontainer-template.json` — metadata.
2. A devcontainer config file — either `.devcontainer.json` at the root or `.devcontainer/devcontainer.json` if the template ships additional files (`Dockerfile`, `docker-compose.yml`, init scripts).

Use `.devcontainer/devcontainer.json` whenever the Template needs more than the JSON file.

## `devcontainer-template.json`

```json
{
  "id": "node-postgres",
  "version": "1.2.0",
  "name": "Node.js + PostgreSQL",
  "description": "Node 22 dev container with a Postgres service.",
  "documentationURL": "https://example.com/templates/node-postgres",
  "licenseURL": "https://example.com/templates/LICENSE",
  "publisher": "Example Org",
  "keywords": ["node", "postgres", "compose"],
  "platforms": ["linux"],
  "options": {
    "nodeVersion": {
      "type": "string",
      "proposals": ["22", "20", "18"],
      "default": "22",
      "description": "Node.js major version."
    },
    "imageVariant": {
      "type": "string",
      "enum": ["bookworm", "bullseye"],
      "default": "bookworm",
      "description": "Debian variant."
    },
    "includeAdminer": {
      "type": "boolean",
      "default": false,
      "description": "Include Adminer for DB inspection."
    }
  },
  "optionalPaths": [
    ".devcontainer/adminer.compose.yml"
  ]
}
```

Key fields:
- `id` — must match the directory name.
- `version` — semver; bump on every change.
- `options` — typed inputs (`boolean` or `string`). `string` supports `enum` (strict) and `proposals` (suggested free-form).
- `platforms` — coarse OS hint for tooling.
- `optionalPaths` — files or directories users may decline. Supporting tools prompt at apply time.

## Placeholders

Use `${templateOption:<name>}` inside template files. Tools substitute the user's chosen value at apply time.

`.devcontainer/devcontainer.json`:

```json
{
  "name": "Node.js + PostgreSQL",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspaces/${localWorkspaceFolderBasename}",
  "features": {
    "ghcr.io/devcontainers/features/node:1": {
      "version": "${templateOption:nodeVersion}"
    }
  }
}
```

`Dockerfile`:

```dockerfile
ARG VARIANT=${templateOption:imageVariant}
FROM mcr.microsoft.com/devcontainers/javascript-node:1-${templateOption:nodeVersion}-${VARIANT}
```

## Authoring rules

- Templates are scaffolds, not runtime configs. Once applied, the user owns the files.
- Use options for choices users would reasonably make at scaffold time (language version, image variant, optional services). Do not over-parameterize.
- Use `enum` for strict values, `proposals` for hints.
- Use `optionalPaths` for files that don't apply universally (e.g. an Adminer Compose override).
- Templates can reference Features in `devcontainer.json` — prefer Features over inline install scripts in a Template's `Dockerfile`.
- Templates and Features live in **separate repositories**. Do not co-locate.
