# Wunder.io AI Agent Guidelines

## 🛡 Environment & Isolation
- **Runtime:** You are running in the `ddev-agents` container as user `vscode`.
- **Root:** Your workspace is `/workspace`.
- **Constraints:** Commands run on Debian Linux/Zsh. Do NOT suggest host-only commands. Use relative paths or `/workspace` — never `/Users/name/...`.
- **No sudo:** The container runs with `no-new-privileges` and all capabilities dropped. Root access is only available from the host via `docker exec -u root`.

## 🏗 Architecture

This repository is a **DDEV add-on** distributed via `ddev addon get wunderio/ddev-agents`. It is not a web application. Its purpose is to provide a standardised, isolated container environment for running agentic AI tools (GitHub Copilot CLI, etc.) inside consumer DDEV projects.

### Key files and their roles

| File | Role |
|---|---|
| `install.yaml` | Add-on manifest — lists files copied into consumer projects on install |
| `docker-compose.agents.yaml` | Defines the `agents` Docker service with security hardening |
| `config.agents.yaml` | DDEV hooks: `pre-start` builds the image, `post-start` runs `devcontainer set-up` |
| `.devcontainer/devcontainer.json` | VS Code/JetBrains attach config — references the DDEV-managed docker-compose |
| `.devcontainer/devcontainer.build.json` | Build-only config — used by `build-devcontainer` to build the image with devcontainer features |
| `copilot-managed-config.json` | Copilot security restrictions (denied tools/files), mounted read-only into the container |
| `commands/host/build-devcontainer` | DDEV host command — builds the devcontainer image via `devcontainer build` |
| `commands/host/set-up` | DDEV host command — applies devcontainer metadata to the already-running container |
| `commands/host/copilot` | DDEV host command — runs `gh copilot` inside the agents container |

### Lifecycle flow

```
ddev start
  → pre-start hook  → build-devcontainer  (builds ddev-${PROJECT}-devcontainer:latest)
  → post-start hook → set-up              (devcontainer set-up --container-id ...)
  → VS Code "Reopen in Container" attaches to the running agents service
```

`ddev copilot` uses `devcontainer exec --container-id` to inject `GH_TOKEN` (from the host's `DDEV_AGENTS_GH_TOKEN`) without storing it in the image or container filesystem.

### Two devcontainer configs

- **`devcontainer.build.json`** — used only during `ddev build-devcontainer`. Defines the base image and features to install.
- **`devcontainer.json`** — used by VS Code/JetBrains to attach. References the DDEV-managed docker-compose stack (`../.ddev/.ddev-docker-compose-full.yaml`).

### Files marked `#ddev-generated`

Files with `#ddev-generated` as the first line are owned by this add-on and will be overwritten when consumers run `ddev addon get wunderio/ddev-agents` again. Do not instruct users to hand-edit these files unless they remove the `#ddev-generated` marker.

## 🔐 Security Conventions

- **Never use `ARG` or `ENV` in Dockerfiles/devcontainer configs for sensitive data** (tokens, passwords, keys). Secrets are injected at runtime via `devcontainer exec` reading `remoteEnv` from `devcontainer.json`.
- The `copilot-managed-config.json` file defines `tools.deny` and `files.deny` lists. Extend these lists rather than relaxing them.
- `SSH_AUTH_SOCK` is explicitly cleared in `remoteEnv` to prevent SSH agent forwarding into the container.

## 🚫 Prohibited Actions
- **NO GIT OPERATIONS:** You do not have git credentials.
    - DO NOT attempt to commit, push, or pull.
    - DO NOT ask the user to run git commands inside the VS Code terminal.
    - Write files to disk; the user handles version control externally.
- **Do not modify `.ddev/` configuration files** unless explicitly asked.