---
name: plugin-test
description: >-
  Acceptance test workflow for Obsidian plugins. Use when asked to test a
  plugin, smoke test a release, verify desktop or mobile behavior, test BRAT
  payloads, inspect runtime errors, or judge whether a UI/editor change works.
targets:
  - "*"
opencode:
  metadata:
    author: obsidian-plugin-development
    version: "1.0"
---

# Obsidian Plugin Acceptance Test

Use this skill for release-level or bug-fix verification of an Obsidian plugin. Keep the bar high: build checks, runtime load, no captured errors, relevant surfaces verified, and desktop plus mobile paths covered unless the plugin is desktop-only.

## Guardrails

- Do not run destructive checks against a production vault unless the user explicitly asks.
- Do not modify source code or commit while executing this skill.
- Reports may be written under `planning/test-reports/`.
- Treat screenshots as evidence, but inspect DOM, runtime state, or console errors before deciding a UI result works.
- Always turn mobile emulation off after mobile checks.
- If any check is skipped, say so in the report.

## Pass 0: Scope

1. Read `manifest.json`, `package.json`, and recent project documentation.
2. Determine plugin id from explicit input or `manifest.json.id`.
3. Identify changed surfaces from the task or recent diff.
4. Build a checklist covering local checks, plugin load, settings, commands, changed surfaces, artifacts, and mobile emulation.

## Pass 1: Local Gate

Run the repository's own checks first. Prefer the package manager implied by lockfiles.

Use the full `check` script when present. Otherwise run available `typecheck`, `lint`, `test`, and `build` scripts.

Record failures exactly. Do not continue to release confidence if the local gate fails unless the user specifically asked for partial diagnosis.

## Pass 2: Runtime Gate

Prefer the repository's real Obsidian verifier when available. Otherwise use Obsidian CLI:

```bash
rtk obsidian plugin:reload id=<plugin-id>
rtk obsidian dev:errors
rtk obsidian eval code="app.plugins.plugins['<plugin-id>'] !== undefined"
```

Verify:

- Plugin loads without captured errors.
- Settings tab opens when the plugin has one.
- Registered commands appear and context-dependent commands use the expected availability.
- Changed reading, editor, view, modal, ribbon, status bar, or settings surfaces work.
- Mobile emulation works unless `manifest.json.isDesktopOnly` is true.

## Pass 3: Mobile Emulation

Use the Obsidian runtime API or CLI equivalent:

```bash
rtk obsidian eval code="app.emulateMobile(true)"
rtk obsidian plugin:reload id=<plugin-id>
rtk obsidian dev:errors
rtk obsidian eval code="app.emulateMobile(false)"
```

Never rely on toggle behavior. Always set an explicit state and reset it.

## Pass 4: Release Payload

For release or BRAT checks, verify the payload directory contains expected assets:

- `main.js`
- `manifest.json`
- `styles.css`
- `versions.json` when preparing community release

Compare `manifest.json.version` with `package.json.version` when both exist.

## Report Shape

```markdown
# Obsidian plugin acceptance test

## Scope
- Plugin id:
- Version:
- Commit:
- Payload:

## Local Gate
- Typecheck:
- Lint:
- Tests:
- Build:

## Runtime Gate
- Desktop reload:
- Desktop errors:
- Desktop surfaces:
- Mobile emulation:
- Mobile errors:
- Mobile surfaces:

## Artifacts
- Payload:
- Version consistency:

## Recommendation
ship / hold, with reason
```

## Failure Modes

- `plugin:reload` can report success even when `onload` throws. Always follow with `dev:errors`.
- Mobile emulation persists until reset.
- A blank screenshot often means the UI did not settle. Inspect DOM and errors before retrying.
- Missing verifier scripts are a harness limitation, not proof the plugin works.
