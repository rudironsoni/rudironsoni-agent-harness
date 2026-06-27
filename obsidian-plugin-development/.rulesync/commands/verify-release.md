---
description: "Run release verification for an Obsidian plugin"
targets: ["*"]
---

plugin_arg = $ARGUMENTS

Default plugin id comes from `manifest.json.id`. If an explicit plugin id is provided, use it.

## Phase 1: Local Gate

Read `manifest.json` and `package.json`, then run the repository's full check script when present:

```bash
rtk bun run check
```

If the repository does not use Bun or has no `check` script, use the detected package manager and run the available `typecheck`, `lint`, `test`, and `build` scripts.

## Phase 2: Runtime Gate

Use the repository's Obsidian runtime verifier when present. Otherwise use Obsidian CLI:

```bash
rtk obsidian plugin:reload id=<plugin-id>
rtk obsidian dev:errors
rtk obsidian eval code="app.plugins.plugins['<plugin-id>'] !== undefined"
```

Verify the settings tab, commands, and changed UI or editor surfaces. If `isDesktopOnly` is not true, also verify mobile emulation with `app.emulateMobile(true)` and reset with `app.emulateMobile(false)`.

## Phase 3: Artifact Verification

Verify release payload expectations:

- `main.js` exists.
- `manifest.json` exists and has valid `id`, `name`, `version`, `minAppVersion`, and `description`.
- `styles.css` exists, even if empty.
- `versions.json` exists for community plugin release readiness.
- `manifest.json.version` matches `package.json.version` when the repository keeps those in sync.
- GitHub or BRAT-style release payloads include only expected plugin assets unless the repository documents additional required files.

## Phase 4: Report

Write or return:

```markdown
# Release Verification Report

## Local Gate
- Full check:
- Build:
- Tests:

## Runtime Gate
- Desktop reload:
- Desktop errors:
- Desktop surfaces:
- Mobile emulation:
- Mobile errors:

## Artifacts
- main.js:
- manifest.json:
- styles.css:
- versions.json:
- version consistency:

## Verdict
ship / hold
```
