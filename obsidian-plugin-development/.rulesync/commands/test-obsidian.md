---
description: "Test an Obsidian plugin on desktop, mobile emulation, or both"
targets: ["*"]
---

target_and_plugin = $ARGUMENTS

Default target is `both`. Default plugin id comes from `manifest.json.id`. If a plugin id is provided, use it.

## Preparation

1. Read `manifest.json` and `package.json`.
2. Determine package manager from lockfiles.
3. If `manifest.json.isDesktopOnly` is true and target includes mobile, mark mobile as skipped with reason.
4. Check for an existing Obsidian runtime or CDP port before launching anything. Reuse the current instance.

## Desktop

1. Run the repository's relevant local check:
   - `rtk bun run check`, `rtk npm run check`, `rtk pnpm run check`, or `rtk yarn check` when present.
   - If no check script exists, run available `typecheck`, `lint`, `test`, and `build` scripts.
2. Reload plugin with Obsidian CLI: `rtk obsidian plugin:reload id=<plugin-id>`.
3. Check `rtk obsidian dev:errors` and console errors.
4. Verify plugin exists: `rtk obsidian eval code="app.plugins.plugins['<plugin-id>'] !== undefined"`.
5. Verify settings, commands, and the changed surface using repository-specific selectors or runtime checks.
6. Capture screenshot only when it helps explain a UI result.

## Mobile

1. Run any repository mobile benchmark or mobile verifier script if present.
2. Enable mobile emulation: `rtk obsidian eval code="app.emulateMobile(true)"`.
3. Reload plugin: `rtk obsidian plugin:reload id=<plugin-id>`.
4. Check `rtk obsidian dev:errors` and console errors.
5. Verify the same changed surfaces that matter for mobile.
6. Always reset: `rtk obsidian eval code="app.emulateMobile(false)"`.

## Report

```markdown
## Test Results
- Target:
- Plugin id:
- Local checks:
- Desktop plugin load:
- Desktop runtime errors:
- Desktop surfaces:
- Mobile emulation:
- Mobile plugin load:
- Mobile runtime errors:
- Mobile surfaces:
- Skipped checks:

Verdict: ship / hold
```
