---
name: devcontainer-author
description: Author, review, and maintain development container configurations, devcontainer Features, devcontainer Templates, and their distribution layouts per the official Dev Container specification. Use when asked to create or modify `.devcontainer/devcontainer.json`, `.devcontainer.json`, `devcontainer-feature.json`, `devcontainer-template.json`, Feature repositories, Template repositories, or OCI publishing layouts. Use for tasks involving containerized development environments, reusable devcontainer features, template scaffolds, feature/template versioning, or publishing to registries such as ghcr.io.
---

# Devcontainer Author

Use this skill to design, review, and modify Dev Container configuration and related artifacts.

## Operating rules

- Prefer the official Dev Container specification over memory or inferred behavior.
- Separate project devcontainer configuration, Features, Templates, and distribution concerns.
- Make surgical changes. Do not rewrite unrelated container files.
- Preserve existing repository style unless it conflicts with the specification.
- Treat generated configuration as source code. Keep it minimal, explicit, and reviewable.
- When changing behavior, explain whether the change affects build time, runtime, lifecycle hooks, user permissions, ports, mounts, or publishing.
- Do not introduce privileged containers, Docker-in-Docker, host mounts, or extra capabilities without explicitly calling out the security tradeoff.

## Task mode

First decide the mode, then route.

- **Authoring** — user wants new or modified files. Output full files (`devcontainer.json`, `devcontainer-feature.json`, `install.sh`, `devcontainer-template.json`, workflow YAML). Use references for canonical shapes.
- **Review** — user wants validation, audit, or feedback on existing files. Output findings against `references/review-checklists.md`, citing offending lines. Do not silently rewrite.

If the request mixes both (e.g. "review and fix"), do the review first, surface findings, then apply only the agreed changes.

## Route by task

- For `.devcontainer/devcontainer.json` or `.devcontainer.json`, read `references/devcontainer-json.md`.
- For authoring a Feature, read `references/features.md`.
- For laying out Feature collections, read `references/features-distribution.md`.
- For authoring a Template, read `references/templates.md`.
- For laying out Template collections, read `references/templates-distribution.md`.
- For CLI usage, GHCR auth, or GitHub Actions publishing, read `references/publishing-workflows.md`.
- For review tasks, also read `references/review-checklists.md`.

## Default workflow

1. Identify the artifact type: devcontainer config, Feature, Feature collection, Template, or Template collection.
2. Inspect existing files before editing.
3. State the minimal intended change.
4. Apply only the necessary changes.
5. Validate against the relevant reference (and the CLI if available — see `publishing-workflows.md`).
6. Report the outcome, including files changed and any unverified assumptions.
