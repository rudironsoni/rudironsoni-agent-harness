---
name: obsidian-cli
description: >-
  Use Obsidian CLI to debug, inspect, and test Obsidian plugins during
  development. Covers plugin reloads, runtime evaluation, console and error
  checks, settings tab checks, DOM inspection, screenshots, CDP, and mobile
  emulation with app.emulateMobile(true).
targets:
  - "*"
opencode:
  metadata:
    author: obsidian-plugin-development
    version: "1.0"
---

# Obsidian CLI

Use this skill to inspect a running Obsidian instance while developing a plugin. Derive the plugin id from explicit input or `manifest.json.id`.

## Essentials

```bash
rtk obsidian plugin:reload id=<plugin-id>
rtk obsidian dev:errors
rtk obsidian dev:console level=error
rtk obsidian commands filter=<plugin-id>
```

`plugin:reload` can return success even when plugin load threw. Always follow with `dev:errors` or `dev:console level=error`.

## Runtime State

```bash
rtk obsidian eval code="app.plugins.plugins['<plugin-id>'] !== undefined"
rtk obsidian eval code="JSON.stringify(app.plugins.plugins['<plugin-id>']?.manifest, null, 2)"
rtk obsidian eval code="app.vault.getName()"
```

Confirm the focused vault before destructive actions.

## Settings And Commands

Open settings when the plugin has a settings tab:

```bash
rtk obsidian eval code="app.setting.open(); app.setting.openTabById('<plugin-id>')"
rtk obsidian dev:screenshot path=planning/test-reports/settings.png
```

Inspect commands:

```bash
rtk obsidian commands filter=<plugin-id>
```

## DOM And Visual Checks

Prefer runtime state and DOM checks before screenshots:

```bash
rtk obsidian dev:dom selector="<selector>" text
rtk obsidian dev:screenshot path=planning/test-reports/surface.png
```

Use selectors from the target repository. Do not import selectors from another plugin unless that repository uses them.

## Mobile Emulation

Use an explicit mobile state and always reset it:

```bash
rtk obsidian eval code="app.emulateMobile(true)"
rtk obsidian plugin:reload id=<plugin-id>
rtk obsidian dev:errors
rtk obsidian dev:screenshot path=planning/test-reports/mobile.png
rtk obsidian eval code="app.emulateMobile(false)"
```

If the CLI provides `dev:mobile`, pass an explicit state rather than relying on toggle behavior.

## CDP Escape Hatch

Prefer `eval`, `commands`, and `dev:dom`. Use CDP only for precise clicks, keyboard events, or state that Obsidian CLI cannot expose:

```bash
rtk obsidian dev:cdp method=Input.dispatchMouseEvent params='{"type":"mousePressed","x":100,"y":200,"button":"left","clickCount":1}'
rtk obsidian dev:cdp method=Input.dispatchMouseEvent params='{"type":"mouseReleased","x":100,"y":200,"button":"left","clickCount":1}'
```

## Resource Rules

- Keep one Obsidian instance.
- Before launching anything, check whether the expected CDP port or helper reports an active instance.
- Reuse the current vault and plugin reload loop.
- Do not create a new vault, new user data directory, or second app process unless the user explicitly asks.
- If a duplicate instance is accidentally launched, stop the duplicate and report it.
