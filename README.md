# DDEV Agents Add-on

Standardized environment for creating an isolated workspace where agentic AI tools can be run safely and consistently across Wunder.io projects.

## Overview

The `ddev-agents` add-on provides a pre-configured `agents` service and VS Code Dev Container settings. Key benefits include:

1.  **Host Isolation**: AI agents run in a dedicated, isolated container. This protects your host machine from potentially errant scripts or external packages executed by AI tools.
2.  **Standardization**: Every project uses the same high-quality, pre-configured environment.
3.  **Maintainability**: Improvements and security updates are pushed centrally through the add-on.

## Workflow & Security

-   **Git Operations**: All Git operations (`commit`, `push`, `pull`) should be performed on your **host machine**. The container provides access to the code, but you should use your host's Git configuration and SSH keys for repository management.
-   **Credential Safety**: ⚠️ **NEVER put any credentials, API keys, or secrets inside the devcontainer.** Always use environment variables or DDEV's built-in secret management to pass necessary keys to the container without storing them in the image or container filesystem.

## Installation

To install the add-on in your DDEV project, run:

```bash
ddev addon get wunderio/ddev-agents
ddev restart
```

## Supported Environments

This add-on is specifically designed and supported for:

-   **VS Code** (Primary interface)
-   **Jetbrains IDEs** (PhpStorm, WebStorm etc.)
-   **Antigravity** (AI Coding Assistant)

### ⚠️ CRITICAL: Connection Check
**Always verify that your IDE is connected to the Dev Container before running any AI agents.** 

If you run agents while still on your host machine, you lose the safety of the isolated environment, and scripts will have direct access to your local system. Look for the "Dev Container: Agents" indicator in the bottom-left corner of VS Code (or your editor's equivalent) before proceeding.

### 1. Start DDEV
Before opening the devcontainer, ensure your DDEV project is running:
```bash
ddev start
```

### 2. Install required plugins
1. For PhpStorm: install the "Dev Containers" plugin:

- Navigate to "Settings → Plugins", enable "Dev Containers" and its dependencies, and restart the IDE


### 3. Open in Editor
1.  Open your project in **VS Code** or **PhpStorm**.
2.  When prompted, click **"Reopen in Container"**
3.  IDE will be relaunched in `agents` container, providing a terminal with all necessary tools pre-installed and running all IDE AI requests inside the container with only access to files inside the `workspace`.

If you don't get the popup you can manually launch the project inside devcontainer:
In VSCode use the Command Palette (< ctrl >-< shift >-p): `Dev Containers: Reopen in Container`.
In PhpStorm:
- Navigate to: "Menu bar → Tools → Services → Dev Containers → Show Dev Containers"
- If there isn't any, click "New Dev Container" under the "Dev Containers" sidebar item
- Under "From Local Project", use the default settings, and from the "Path to devcontainer.json", select "Specify Path" and the path of your project's devcontainer.json (for example: /Users/myuser/Sites/myproject/.devcontainer/devcontainer.json)


## Features

-   **Isolated Python Environment**: Based on `mcr.microsoft.com/devcontainers/python:3-bookworm`.
-   **Integrated Tools**: Pre-installed Node.js, GH CLI, Git and common utilities.
-   **GitHub Copilot (Agent Mode)**: Includes the `copilot` CLI extension.
-   **Secure Authentication**: Uses your host's `DDEV_AGENTS_GH_TOKEN` automatically, so you never have to type credentials inside the container.

## GitHub Authentication (Recommended Setup)

To use GitHub Copilot (Agent Mode) or `gh` commands without repetitive logins, set up a Personal Access Token (PAT) on your **host machine**.

### 1. Generate a GitHub Token
1.  Go to [GitHub Fine-grained Tokens](https://github.com/settings/personal-access-tokens/new).
2.  **Name & Expiration**: Set a name (e.g., "Copilot CLI Local") and a reasonable expiration (max 366 days).
3.  **Repository Access**: Select **Public Repositories (read-only)** or **Only select repositories** (even if you select none). The CLI works on local files and doesn't need to see your remote repos.
4.  **Account Permissions**:
    -   Find the **Copilot Requests** dropdown.
    -   Select **Access: Read-only**.
5.  Generate and copy the token.

### 2. Configure your Host Machine
Add the token to your shell profile so it's always available when you start the devcontainer.

**For macOS (Zsh or Bash):**
1.  Open your shell profile (e.g., `~/.zshrc` or `~/.profile`):
    ```bash
    nano ~/.zshrc
    ```
2.  Add this line at the end:
    ```bash
    export DDEV_AGENTS_GH_TOKEN=your_token_here
    ```
3.  Save and restart your terminal (or run `source ~/.zshrc`).

**For Linux/Ubuntu:**
1.  Run 
    ```bash
    systemctl --user edit --full --force environment.d/myenv.conf`
    ```
2.  Add line:
    ```bash
    DDEV_AGENTS_GH_TOKEN=your_token_here
    ```
3.  Save and run
    ```bash
    systemctl --user daemon-reload
    ```


### 3. Benefits
-   **Zero-Interaction**: The devcontainer automatically picks up host's `$DDEV_AGENTS_GH_TOKEN` and uses it as `$GH_TOKEN` inside the container, then configures the `gh-copilot` extension.
-   **Security**: No credentials are ever typed or stored inside the container's history/filesystem. 

## Centralized Development

The purpose of this repository is to serve as the single source of truth for our agentic workflows. By using `ddev addon get`, projects can stay up-to-date with our latest agent configurations by simply running `ddev addon get wunderio/ddev-agents` again.

## Troubleshooting

*General:*

Container (re)build fails with the following error message:
```
W: GPG error: [https://dl.yarnpkg.com/debian](vscode-file://vscode-app/Applications/Visual%20Studio%20Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) stable InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 62D54FD4003F6525
E: The repository '[https://dl.yarnpkg.com/debian](vscode-file://vscode-app/Applications/Visual%20Studio%20Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) stable InRelease' is not signed.
ERROR: Feature "Common Utilities" (ghcr.io/devcontainers/features/common-utils) failed to install!
```
> There was an issue in earlier versions of devcontainer with expired Yarn GPG keys. This has already been fixed upstream, but if you are still getting the erro you probably have those older images cached on your local machine.
To fix the issue you need to remove the old images from your system by running the following commands:
`docker images -a` <- find the devcontainer image id
`docker rmi mcr.microsoft.com/devcontainers/python:3-bookworm`

**sudo not working inside the container:**
> `sudo` is intentionally disabled via `no-new-privileges` for security hardening.
> If you need root access, run from your host terminal:
> `ddev ssh -s agents -u root`

**Troubleshooting for macOS**:

$GH_TOKEN is not recognized after rebuilding
> Try launching VS Code directly from your terminal by running `code .` in your project folder. This ensures VS Code inherits your shell's environment variables.

**Troubleshooting for Linux/Ubuntu:**

If you get error `ERROR: unable to prepare context: path "/tmp/devcontainercli-something/empty-folder" not found` 
>Make sure you don't have Docker installed as a snap package. If it is, remove the snap and install Docker the traditional way

(PhpStorm) If you get error `ERROR [stage-0 4/7] RUN chmod -R 0755 /tmp/jb-devcontainer-features/ghcr.io-devcontainers-features-node-1 && cd /tmp/jb-devcontainer-features/ghcr.io-devcontainers-features-node-1 && chmod +x ./devconta  40.6s`
>You need to add
> ```bash
>    {
>      "dns": ["8.8.8.8", "8.8.4.4", "1.1.1.1"]
>    }
>    ```
>    Into your `/etc/docker/daemon.json` file and restart docker (`sudo systemctl restart docker`)


