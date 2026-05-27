# `devcontainer.json` authoring

`devcontainer.json` holds the metadata and settings needed to configure a development container for a runtime stack or tool. It lives at `.devcontainer/devcontainer.json` (preferred for multi-file setups) or `.devcontainer.json` at the repo root.

## Metadata merging

Some metadata properties can also be embedded in the image via the `devcontainer.metadata` label. At runtime, the label and `devcontainer.json` are merged. When authoring, prefer `devcontainer.json` unless a property must travel with the image.

## Primary scenarios (pick exactly one)

### Image-based

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu"
}
```

### Dockerfile-based

```json
{
  "build": {
    "dockerfile": "Dockerfile",
    "context": "..",
    "args": { "VARIANT": "22-bookworm" }
  }
}
```

### Docker Compose-based

```json
{
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "runServices": ["db"],
  "workspaceFolder": "/workspaces/${localWorkspaceFolderBasename}"
}
```

Use `runServices` to limit which Compose services start with the dev container.

## Common properties

| Property | Notes |
|---|---|
| `workspaceMount`, `workspaceFolder` | Override default workspace bind. Compose mode requires `workspaceFolder` only. |
| `runArgs` | Extra `docker run` flags. Avoid `--privileged`, `--cap-add`, broad mounts. |
| `forwardPorts` | Preferred over legacy `appPort`. Accepts numbers or `"host:port"`. |
| `portsAttributes`, `otherPortsAttributes` | Label and configure forwarded ports (onAutoForward, protocol, etc.). |
| `containerEnv`, `remoteEnv` | `containerEnv` is set on the container; `remoteEnv` only for the connected tool/process. |
| `containerUser`, `remoteUser` | `containerUser` affects all in-container operations. `remoteUser` only affects the connected tool and its subprocesses. |
| `mounts` | Bind/volume mounts. Use sparingly; justify host paths. |
| `features` | Map of Feature id → options. See `features.md`. |
| `overrideFeatureInstallOrder` | Forces Feature install order. See `features.md`. |
| `customizations` | Tool-specific settings (e.g. `vscode.extensions`). |

## Lifecycle commands

Run on host or in container at distinct phases. Choose the earliest phase that satisfies the need.

| Command | Where | When | Typical use |
|---|---|---|---|
| `initializeCommand` | Host | Before container create | Pre-clone/setup steps that must run on the host |
| `onCreateCommand` | Container | Once, during container creation | Heavy setup baked into the image-or-volume lifecycle |
| `updateContentCommand` | Container | After source mount/update | Restore packages, codegen tied to source |
| `postCreateCommand` | Container | After create, once per container | First-time project setup |
| `postStartCommand` | Container | Every container start | Services, daemons |
| `postAttachCommand` | Container | Every tool attach | Per-attach hints, smoke checks |

### Command forms

- **String** — runs via shell (`/bin/sh -c`). Use when you need globs, pipes, env expansion.
  ```json
  "postCreateCommand": "npm ci && npm run build"
  ```
- **Array** — exec form, no shell parsing. Safer when arguments contain spaces.
  ```json
  "postCreateCommand": ["bash", "-lc", "npm ci"]
  ```
- **Object** — keyed steps run in parallel.
  ```json
  "postCreateCommand": {
    "install": "npm ci",
    "fetch":   "git fetch --all"
  }
  ```

## Authoring rules

- Pick one primary scenario. Do not mix `image` and `build`.
- Reach for `forwardPorts` first; only use `appPort` for legacy publish behavior.
- Put reusable tooling in Features, not lifecycle scripts. Put project bootstrap (deps, codegen) in lifecycle commands.
- Set `remoteUser` to a non-root user when the image supports it.
- Never embed secrets. Use `remoteEnv` referencing host env vars, or external secret stores.
- For Compose, keep app config in `docker-compose.yml`; keep dev-only overrides in `devcontainer.json` or a Compose override file.
