# Publishing workflows

This reference covers the `devcontainer` CLI, GHCR authentication, and GitHub Actions publishing for Feature and Template collections.

## Dev Container CLI

Install:

```sh
npm install -g @devcontainers/cli
```

Common commands:

| Command | Purpose |
|---|---|
| `devcontainer build --workspace-folder .` | Build the dev container image from `devcontainer.json`. |
| `devcontainer up --workspace-folder .` | Start (and create if needed) the dev container. |
| `devcontainer exec --workspace-folder . <cmd>` | Run a command inside the dev container. |
| `devcontainer features publish --namespace <ns> ./src` | Package and push all Features in `./src` to `<registry>/<ns>/<id>`. |
| `devcontainer features test --features <id> --base-image <img> ./` | Run Feature tests from `test/<id>/test.sh`. |
| `devcontainer features info <oci-ref>` | Inspect a published Feature. |
| `devcontainer templates publish --namespace <ns> ./src` | Package and push all Templates in `./src`. |
| `devcontainer templates apply --template-id <oci-ref> --template-args '{}' --workspace-folder ./out` | Scaffold a Template into a target folder. |
| `devcontainer templates metadata <oci-ref>` | Inspect a published Template. |

The `--namespace` flag is the OCI namespace under the registry, e.g. `owner/features` for `ghcr.io/owner/features/...`. Set the registry via `--registry` (defaults to `ghcr.io`).

## GHCR authentication

For local pushes:

```sh
echo "$GITHUB_TOKEN" | docker login ghcr.io -u <username> --password-stdin
# or:
gh auth token | docker login ghcr.io -u <username> --password-stdin
```

Required token scopes: `write:packages`, `read:packages`. Use a fine-grained PAT or the `GITHUB_TOKEN` injected into GitHub Actions runs.

After the first push, set the package's visibility under **Packages → Settings** on GitHub (public is typical for OSS Features/Templates).

## GitHub Actions: publish Features

Canonical workflow using the official action:

```yaml
name: Release Features

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: devcontainers/action@v1
        with:
          publish-features: "true"
          base-path-to-features: "./src"
          generate-docs: "true"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

The action:
- Iterates `./src/<feature-id>/`.
- Packages each Feature as `devcontainer-feature-<id>.tgz`.
- Pushes to `ghcr.io/<owner>/<repo>/<feature-id>` at semver tags `X`, `X.Y`, `X.Y.Z`, and `latest`.
- Optionally regenerates `README.md` per Feature.

## GitHub Actions: test Features

```yaml
name: Test Features

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        feature: [feature-a, feature-b]
        base-image:
          - mcr.microsoft.com/devcontainers/base:ubuntu
          - mcr.microsoft.com/devcontainers/base:debian
    steps:
      - uses: actions/checkout@v4
      - uses: devcontainers/action@v1
        with:
          features-test-mode: "true"
          features-test-base-image: ${{ matrix.base-image }}
          features-test-filter: ${{ matrix.feature }}
```

## GitHub Actions: publish Templates

```yaml
name: Release Templates

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: devcontainers/action@v1
        with:
          publish-templates: "true"
          base-path-to-templates: "./src"
          generate-docs: "true"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Image labels and metadata

When publishing dev container images (not Features/Templates), set the `devcontainer.metadata` label to embed selected `devcontainer.json` properties in the image. Tools merge that label with `devcontainer.json` at runtime — useful for shipping defaults inside a base image.

## Visibility and consumption

- After publishing, OCI artifacts default to **private** on GHCR. Mark them public if consumers outside the org should pull them.
- Consumers reference Features/Templates by their full OCI path; no registry-side index is required.
- Bumping a Feature's `version` republishes the new tags; the floating `:1` and `:latest` tags advance automatically.
