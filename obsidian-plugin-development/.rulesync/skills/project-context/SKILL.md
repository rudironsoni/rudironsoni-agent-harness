---
name: project-context
description: "Summarize an Obsidian plugin project context and validation constraints"
targets: ["*"]
---

# Project Context

Use this skill when starting work in an Obsidian plugin repository or handing off context to another agent.

## What To Inspect

- `manifest.json`: plugin id, name, version, `minAppVersion`, `isDesktopOnly`, and release metadata.
- `package.json`: package manager, scripts, test commands, build commands, release commands, and Obsidian-related dependencies.
- Source entry points: plugin `onload`, settings tab, registered commands, views, editor extensions, markdown processors, and any mobile or desktop guards.
- Test and verifier scripts: unit tests, typechecks, lint, startup benchmarks, Obsidian runtime verifiers, and release artifact checks.
- Runtime harness files: test vault location, CDP port, Obsidian CLI usage, mobile emulation helpers, screenshot output, and report directories.
- Release outputs: `main.js`, `manifest.json`, `styles.css`, `versions.json`, and any packaged assets expected by the repository.

## Summary Format

Return a concise summary with:

- Plugin identity and target platforms.
- Important plugin surfaces and architecture boundaries.
- Local development commands and package manager.
- Runtime verification path for desktop and mobile.
- Release or BRAT payload expectations.
- Current uncertainty, skipped checks, and assumptions.

Do not invent missing scripts or surfaces. If a target repository lacks a verifier, say so and propose the smallest runtime check that can confirm the changed behavior.
