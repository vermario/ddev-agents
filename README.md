# DDEV Agents Add-on

Standardized environment for creating an isolated workspace where agentic AI tools can be run safely and consistently across Wunder.io projects.

## Overview

The `ddev-agents` add-on provides a pre-configured `agents` service and VS Code Dev Container settings. Key benefits include:

1.  **Host Isolation**: AI agents run in a dedicated, isolated container. This protects your host machine from potentially errant scripts or external packages executed by AI tools.
2.  **Standardization**: Every project uses the same high-quality, pre-configured environment.
3.  **Maintainability**: Improvements and security updates are pushed centrally through the add-on.

## Getting Started

This section walks through the complete setup from scratch. Follow each step in order.

### Prerequisites

Before you begin, ensure you have:

-   [DDEV](https://ddev.readthedocs.io/en/stable/) installed on your host machine
-   An active **GitHub Copilot subscription** (Individual, Business, or Enterprise)
-   **VS Code** or a **JetBrains IDE** (PhpStorm, WebStorm etc.)

### Step 1: Install the Add-on

In your DDEV project directory, run:

```bash
ddev addon get wunderio/ddev-agents
ddev restart
```

### Step 2: Set Up GitHub Authentication

To use GitHub Copilot and `gh` commands without repetitive logins, create a Personal Access Token (PAT) and add it to your host machine's environment.

**Generate a GitHub token:**

1.  Go to [GitHub Fine-grained Tokens](https://github.com/settings/personal-access-tokens/new).
2.  **Name & Expiration**: Set a name (e.g., "Copilot CLI Local") and a reasonable expiration (max 366 days).
3.  **Repository Access**: Select **Public Repositories (read-only)** or **Only select repositories** (even if you select none). The CLI works on local files and doesn't need to see your remote repos.
4.  **Account Permissions**: Find the **Copilot Requests** dropdown and select **Access: Read-only**.
5.  Generate and copy the token.

**Add the token to your shell profile:**

1.  Open your shell profile (e.g., `~/.zshrc` for Zsh, `~/.bashrc` for Bash):
    ```bash
    nano ~/.zshrc
    ```
2.  Add this line at the end:
    ```bash
    export DDEV_AGENTS_GH_TOKEN=your_token_here
    ```
3.  Save and reload your profile:
    ```bash
    source ~/.zshrc
    ```

The devcontainer automatically picks up `$DDEV_AGENTS_GH_TOKEN` from your host and uses it as `$GH_TOKEN` inside the container — no credentials are ever typed or stored inside the container.

### Step 3: Set Up the WQS API Key (Recommended)

The [Wunder Quality System](https://quality.wunder.io) MCP server gives AI agents access to Wunder's quality standards and guidelines. The API key is stored in **LastPass** — search for:
> **Wunder Quality System MCP API key**

1.  Open your shell profile (same file as above):
    ```bash
    nano ~/.zshrc
    ```
2.  Add this line at the end:
    ```bash
    export WQS_MCP_API_KEY=your_key_here
    ```
3.  Save and reload your profile:
    ```bash
    source ~/.zshrc
    ```

### Step 4: Start DDEV

```bash
ddev start
```

### Step 5: Open in Your Editor

#### VS Code

1.  Open your project folder in VS Code.
2.  When prompted, click **"Reopen in Container"**, or open the Command Palette (`Ctrl`+`Shift`+`P`) and run `Dev Containers: Reopen in Container`.
3.  VS Code will relaunch inside the `agents` container with all tools pre-installed.

#### PhpStorm / JetBrains IDEs

1.  Install the **Dev Containers** plugin: navigate to **Settings → Plugins**, enable "Dev Containers" and its dependencies, then restart the IDE.
2.  Navigate to **Menu bar → Tools → Services → Dev Containers → Show Dev Containers**.
3.  If no container is listed, click **New Dev Container**, select **From Local Project** (keep the default source settings), choose **Specify Path** for the devcontainer.json, and point it to your project's `.devcontainer/devcontainer.json` (e.g. `/Users/myuser/Sites/myproject/.devcontainer/devcontainer.json`).

### ⚠️ CRITICAL: Connection Check

**Always verify that your IDE is connected to the Dev Container before running any AI agents.**

If you run agents while still on your host machine, you lose the safety of the isolated environment, and scripts will have direct access to your local system. Look for the **"Dev Container: Agents"** indicator in the bottom-left corner of VS Code (or your editor's equivalent) before proceeding.

## Features

-   **Isolated Python Environment**: Based on `mcr.microsoft.com/devcontainers/python:3-bookworm`.
-   **Integrated Tools**: Pre-installed Node.js, GH CLI, Git and common utilities.
-   **Dynamic Versions**: PHP and Node.js versions are read from `.ddev/config.yaml` and injected at build time.
-   **SSH-Based Tool Execution**: MCP tools execute commands in the DDEV web container via SSH (no Docker socket access).
-   **MCP Tool Configs**: Pre-configured tools for Drush, Composer, Drupal MCP proxy, and log access (`.agents/tools-config/`).
-   **GitHub Copilot (Agent Mode)**: Includes the `copilot` CLI extension.
-   **Secure Authentication**: Uses your host's `DDEV_AGENTS_GH_TOKEN` automatically, so you never have to type credentials inside the container.
-   **Wunder Quality System MCP**: Pre-configured access to Wunder's quality standards and guidelines.

## Architecture

The agents container communicates with the DDEV web container via SSH:

-   **Ephemeral SSH Keys**: Ed25519 keypair generated fresh on every `ddev start`, distributed via `docker exec`
-   **Automatic User Detection**: The DDEV user is detected from `/var/www/html` ownership in the web container
-   **No Docker Socket**: The agents container has no access to the Docker daemon — all command execution goes through SSH
-   **Security Hardened**: All Linux capabilities dropped, privilege escalation disabled

### MCP Tools

The `.agents/tools-config/` directory contains YAML definitions for tools exposed via the [wdrmcp](https://github.com/wunderio/wdrmcp) MCP server:

-   **Drush**: Site management, cache rebuild, site install, PHP eval, watchdog logs
-   **Composer**: Install, update, require dependencies
-   **Logs**: Nginx access/error logs, PHP-FPM logs
-   **Drupal MCP**: Proxy to Drupal's built-in MCP server endpoint

See `.agents/README.md` for details on adding custom tools and configuring credentials.

## GitHub Copilot CLI

Run GitHub Copilot CLI directly from your host terminal via `ddev copilot`, with all execution happening inside the isolated agents container.

### Usage

**Interactive mode** (REPL):
```bash
ddev copilot
```

**Headless mode** (single prompt):
```bash
ddev copilot -p "explain what this function does"
```

**Pass any Copilot CLI flags**:
```bash
ddev copilot --version
ddev copilot --help
```

Verify your setup by running `/status` in the Copilot REPL.

### Installing the devcontainer CLI

`ddev copilot` requires the [devcontainer CLI](https://github.com/devcontainers/cli) on your host machine. The command will attempt to auto-install it via `npm` on first run, or fall back to `npx` — but for the fastest startup, install it once globally:

**npm (recommended):**
```bash
npm install -g @devcontainers/cli
```

**Homebrew (macOS):**
```bash
brew install devcontainer
```

**Winget (Windows):**
```bash
winget install Microsoft.DevContainerCLI
```

> If you have Node.js / npm but not the CLI, `ddev copilot` will install it automatically on the first run. If `npx` is available but not `npm`, it will use `npx` as a one-shot runner (slower on first invocation). If none of these are present, the command exits with clear instructions.

### How It Works

-   **devcontainer CLI**: `ddev copilot` uses `devcontainer exec --container-id` to run inside the DDEV-managed container. `--container-id` bypasses devcontainer's own container-discovery (which only finds containers started via `devcontainer up`), while still reading `devcontainer.json` to inject `remoteEnv` — including `GH_TOKEN` from your host's `DDEV_AGENTS_GH_TOKEN`
-   **Pre-installed**: Copilot CLI is installed via devcontainer feature during container build
-   **Persistent state**: Configuration is stored in a Docker volume (`ddev-${DDEV_PROJECT}-copilot-state`)
-   **Project-specific state**: Each DDEV project has its own isolated configuration volume
-   **State survives rebuilds**: Configuration survives container rebuilds (`ddev restart`)
-   **Authentication**: Uses the `GH_TOKEN` environment variable injected via devcontainer CLI from your host's `DDEV_AGENTS_GH_TOKEN`
-   **Always up-to-date**: Copilot CLI is reinstalled during container rebuilds to ensure latest version

### Security

-   **Token-based auth**: Uses your GitHub token from the host environment (never stored in the container)
-   **Managed configuration**: Security restrictions (`copilot-managed-config.json`) are mounted read-only into the container
-   **Container isolation**: Copilot runs inside the agents container with all security hardening (no capabilities, no privilege escalation)
-   **Denied operations**: Git destructive operations, publishing packages, SSH access, and reading sensitive files are blocked

### Resetting Copilot State

To clear all configuration data for the current project:
```bash
# Replace ${DDEV_PROJECT} with your project name (from .ddev/config.yaml)
docker volume rm ddev-${DDEV_PROJECT}-copilot-state
```
Copilot will re-initialize on the next run.

## Wunder Quality System MCP

The [Wunder Quality System](https://quality.wunder.io) (WQS) is Wunder's internal knowledge base of quality standards, best practices, and project guidelines. The WQS MCP server exposes this knowledge directly to AI agents, allowing them to reference Wunder's standards when generating code, writing documentation, or making architectural decisions.

The `wunder-quality-system` MCP server is automatically configured in both VS Code and Copilot CLI during `ddev set-up`. It connects to `https://quality.wunder.io/mcp` using a shared API key (see [Step 3](#step-3-set-up-the-wqs-api-key-recommended) in Getting Started).

## Workflow & Security

-   **Git Operations**: All Git operations (`commit`, `push`, `pull`) should be performed on your **host machine**. The container provides access to the code, but you should use your host's Git configuration and SSH keys for repository management.
-   **Credential Safety**: ⚠️ **NEVER put any credentials, API keys, or secrets inside the devcontainer.** Always use environment variables or DDEV's built-in secret management to pass necessary keys to the container without storing them in the image or container filesystem.

## Centralized Development

The purpose of this repository is to serve as the single source of truth for our agentic workflows. By using `ddev addon get`, projects can stay up-to-date with our latest agent configurations by simply running `ddev addon get wunderio/ddev-agents` again.

## Troubleshooting

**Authentication fails (Copilot CLI):**
> Ensure `DDEV_AGENTS_GH_TOKEN` is set on your host and includes "Copilot Requests: Read-only" permission. Restart your terminal and run `ddev restart` after setting the token.

**WQS MCP server not connecting:**
> Ensure `WQS_MCP_API_KEY` is set on your host machine (see [Step 3](#step-3-set-up-the-wqs-api-key-recommended)). Restart your terminal and run `ddev restart`.

**Container (re)build fails with a GPG/Yarn key error:**
```
W: GPG error: https://dl.yarnpkg.com/debian stable InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 62D54FD4003F6525
E: The repository 'https://dl.yarnpkg.com/debian stable InRelease' is not signed.
ERROR: Feature "Common Utilities" (ghcr.io/devcontainers/features/common-utils) failed to install!
```
> There was an issue in earlier versions of devcontainer with expired Yarn GPG keys. This has been fixed upstream, but if you still see the error you probably have older images cached locally. Remove them with:
> ```bash
> docker images -a   # locate mcr.microsoft.com/devcontainers/python:3-bookworm in the list
> docker rmi mcr.microsoft.com/devcontainers/python:3-bookworm
> ```

**sudo not working inside the container:**
> `sudo` is intentionally disabled via `no-new-privileges` for security hardening.
> If you need root access, run from your host terminal:
> `ddev ssh -s agents -u root`

**macOS: `$GH_TOKEN` is not recognized after rebuilding:**
> Try launching VS Code directly from your terminal by running `code .` in your project folder. This ensures VS Code inherits your shell's environment variables.

**Linux/Ubuntu: `ERROR: unable to prepare context: path "/tmp/devcontainercli-something/empty-folder" not found`:**
> Make sure Docker is not installed as a snap package. If it is, remove the snap and install Docker the traditional way.

**PhpStorm: `ERROR [stage-0 4/7] RUN chmod -R 0755 /tmp/jb-devcontainer-features/...`:**
> Add the following to your `/etc/docker/daemon.json` and restart Docker (`sudo systemctl restart docker`):
> ```json
> {
>   "dns": ["8.8.8.8", "8.8.4.4", "1.1.1.1"]
> }
> ```


