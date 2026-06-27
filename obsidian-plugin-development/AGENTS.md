# Obsidian Plugin Development Harness

Use this package for Obsidian plugin development, debugging, testing, review, and release maintenance. Prefer repository-specific scripts and architecture when present. When details differ between this harness and the target plugin repository, inspect the target repository first and follow its source of truth.

## Runtime Verification

- Do not claim UI or runtime bugs are fixed from unit tests, typechecks, or builds alone. Run the relevant Obsidian runtime check, or state clearly that runtime verification was skipped.
- Keep one Obsidian instance. Before launching or spawning a helper, check the active CDP port or the repository helper that detects a running instance. Reuse the existing instance and prefer plugin reload over relaunch.
- Use `manifest.json.id` as the default plugin id. If a command accepts an explicit plugin id, the explicit value wins.
- Desktop and mobile behavior both matter unless `manifest.json` has `"isDesktopOnly": true`. For mobile checks, use `app.emulateMobile(true)`, reload the plugin, verify behavior, and always reset with `app.emulateMobile(false)`.
- Screenshots are useful evidence, but inspect DOM state and console errors before calling a visual result correct.

## Obsidian Plugin Rules

- Use `this.app`, not the global `app`, in plugin code.
- Use `requestUrl()` instead of `fetch()` for plugin network requests.
- Register cleanup through `registerEvent()`, `registerDomEvent()`, `registerInterval()`, `addCommand()`, `addSettingTab()`, and `registerView()` where applicable.
- Do not use unsafe `innerHTML` or `outerHTML` for user-controlled content. Use Obsidian DOM helpers, `setText()`, or safe rendering APIs.
- Use `vault.configDir` rather than hardcoded `.obsidian` paths.
- Use `fileManager.trashFile()` for user file deletion flows when appropriate.
- Avoid regex lookbehind because older mobile engines can break on it.
- Keep icon-only controls keyboard accessible and give them accessible labels.
- Use Obsidian CSS variables and test light, dark, desktop, and mobile layouts when UI or CSS changes.

## Change Discipline

- Touch only files required for the task. Do not refactor unrelated plugin architecture or generated files.
- Preserve target repository conventions for package manager, build tools, tests, and release scripts.
- Treat generated release artifacts as outputs. Update source and regenerate artifacts through the target repository workflow.
- Keep credentials and local vault paths out of committed files. Use environment variables or ignored local files for secrets.
