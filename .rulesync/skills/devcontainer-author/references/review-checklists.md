# Devcontainer review checklist

Use this when the task mode is **review**. Walk the relevant checklists, cite file paths and line numbers for each finding, and group findings by severity (block / suggest / nit).

## Artifact classification

- Is this a project devcontainer, Feature, Template, Feature collection, or Template collection?
- Is the chosen artifact type appropriate for the requested behavior?
  - Reusable tooling → Feature.
  - Project-specific config → `devcontainer.json` + lifecycle commands.
  - Scaffold for new projects → Template.

## `devcontainer.json`

- Uses exactly one primary scenario unless intentionally composed: `image`, `build`, or `dockerComposeFile`.
- Uses `forwardPorts` instead of `appPort` unless publishing behavior is required.
- Sets `remoteUser` and `containerUser` deliberately; non-root where the image supports it.
- Avoids `privileged`, `--cap-add`, broad mounts, and host access unless justified.
- Places repeatable environment setup in image, Dockerfile, Feature, or lifecycle hook **according to scope**:
  - Image / Dockerfile: stable, shared, expensive to install.
  - Feature: reusable across projects.
  - `postCreateCommand`: project-specific, runs once.
  - `postStartCommand`: must run every start (daemons).
- Uses array command form when shell parsing is not needed.
- Uses string command form only when shell behavior is intentional.
- Uses object command form when steps are independent and should run in parallel.
- Does not put secrets directly in config. Uses `remoteEnv` referencing host vars.
- `features` block uses pinned major or minor tags (`:1`, `:1.2`), not unpinned references.
- `overrideFeatureInstallOrder` is used only when necessary; prefers `installsAfter` in the Feature itself.

## Feature

- Directory name matches `devcontainer-feature.json.id`.
- Includes `devcontainer-feature.json` and `install.sh`.
- `version` is semver and bumped on every behavior change.
- Installs reusable tooling, not project-specific application dependencies.
- `install.sh` is idempotent and distro-aware (does not assume a single base image).
- Cleans up package caches.
- Declares meaningful `options` with sensible defaults; uses `enum` vs `proposals` correctly.
- Declares `installsAfter` when ordering matters.
- Has tests under `test/<id>/test.sh` when behavior is non-trivial.
- Does not request elevated capabilities (`privileged`, `capAdd`, `securityOpt`) without documented justification.

## Feature distribution

- Layout is `src/<feature-id>/...`.
- Tests mirror `src/` under `test/<feature-id>/test.sh`.
- Tarballs are named `devcontainer-feature-<id>.tgz`.
- OCI naming follows `<registry>/<namespace>/<id>[:version]`.
- Publishing produces `devcontainer-collection.json` (generated, not hand-edited).
- Repository does **not** mix Templates and Features.
- CI publishes via `devcontainers/action@v1` with `publish-features: true`.
- GHCR package visibility is intentional (public vs private).

## Template

- Directory name matches `devcontainer-template.json.id`.
- Includes `devcontainer-template.json`.
- Includes `.devcontainer.json` or `.devcontainer/devcontainer.json`. Uses the directory form when shipping additional files.
- `version` is semver and bumped on every change.
- Uses Template `options` only for meaningful user choices; not over-parameterized.
- Uses `enum` for strict values and `proposals` for suggested free-form values.
- Uses `optionalPaths` for files users may reasonably decline.
- Prefers referencing Features over inline install steps in Dockerfiles.
- Placeholders `${templateOption:<name>}` resolve cleanly; no stray placeholders in non-template files.

## Template distribution

- Layout is `src/<template-id>/...`.
- Tarballs are named `devcontainer-template-<id>.tgz`.
- OCI naming follows `<registry>/<namespace>/<id>[:version]`.
- Template namespace is distinct from any Feature collection namespace.
- Templates and Features are in **separate** repositories.
- CI publishes via `devcontainers/action@v1` with `publish-templates: true`.

## Reporting

For each finding:
1. File path and line range.
2. Severity: **block** (spec violation, security issue), **suggest** (best practice), **nit** (style).
3. The rule from this checklist that applies.
4. A concrete remediation — a snippet or edit, not a description.

Do not silently rewrite files in review mode. Surface findings and wait for the user to approve fixes.
