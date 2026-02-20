# Local Development MCP

An MCP server that gives AI agents tools to run commands in DDEV containers via SSH. Tools are defined in YAML configuration files.

## Architecture

- **Execution Method**: SSH (passwordless key-based authentication)
- **SSH Keys**: Ephemeral Ed25519 keys, generated fresh each `ddev start` and distributed via `docker exec`
- **SSH User**: Automatically detected from web container's `/var/www/html` ownership, configured via SSH client `User` directive
- **No Docker Socket**: Fully isolated from host Docker daemon
- **No Persistent State**: No `.runtime.env` or stored keys — everything is set up by the `set-up` hook

## How to Use

1. **Installation:**
   ```bash
   cd /path/to/ddev-project
   ddev get wunderio/ddev-agents
   ddev restart  # Builds container and sets up SSH automatically
   ```

2. **First Launch:**
   - Open VS Code in the devcontainer
   - The wdrmcp MCP server config is at `.vscode/mcp.json`
   - Note: Due to VS Code bug, search `@mcp` in extension gallery to enable the MCP registry
   - Open Command Palette: `MCP Servers: List`, find wdrmcp and click Start

3. **Using Tools:**
   - Open VS Code Copilot chat
   - Use tools like `drush`, `composer_install`, `logs_nginx_access`, etc.
   - All tools connect via SSH automatically

## How SSH Works

SSH keys are **ephemeral** — generated fresh on every `ddev start`:

1. The `set-up` hook (runs post-start on host) generates an Ed25519 keypair
2. Public key is placed in the web container via `docker exec`
3. Private key + SSH client config are placed in the agents container via `docker exec`
4. The SSH client config includes a `User` directive with the detected DDEV user
5. Keys exist only in container memory — never written to disk on the host

Tool configs use `ssh_target: "web"` — the SSH client config handles which user to connect as.

## Files

- `tools-config/` – YAML tool definitions
- `mcp.json` – MCP server configuration (copied to `.vscode/mcp.json` by set-up hook)

## Adding a Tool

Create a file in `tools-config/my_tool.yml`:

```yaml
tools:
  - name: my_tool
    type: command
    enabled: true
    description: "What this tool does"
    command_template: "my_command {arg1}"
    ssh_target: "web"
    working_dir: "/var/www/html"
    input_schema:
      type: object
      properties:
        arg1:
          type: string
          description: "Argument description"
      required:
        - arg1
```

## Project `.env` Credentials

If any MCP tool config needs credentials (e.g., Drupal MCP proxy), define them in `.env` inside the `tools-config` directory.

Example credentials file (`.agents/tools-config/.env`):

```dotenv
DRUPAL_MCP_USER=admin
DRUPAL_MCP_PASS=admin
```

This file is preserved across addon reinstalls (non-destructive merge).

## Available Tool Types

- `command` – Run shell commands with parameter substitution
- `mcp_server` – Proxy to additional internal MCP servers

### Command Tool Type

Command tools execute shell commands in DDEV containers with parameter substitution.

Example:

```yaml
tools:
  - name: my_command_tool
    type: command
    enabled: true
    description: "Execute a custom command"
    command_template: "my_command {arg1}"
    ssh_target: "web"
    working_dir: "/var/www/html"

    input_schema:
      type: object
      properties:
        arg1:
          type: string
      required:
        - arg1
```

### MCP Server Tool Type

MCP Server tools proxy requests to other MCP servers via HTTP. This allows the Python MCP server to act as a gateway to other specialized MCP servers.
Both plain JSON-RPC HTTP endpoints and Streamable HTTP MCP endpoints are supported.

**Dynamic Tool Discovery:** When `expose_remote_tools: true` is set, the proxy will query the remote MCP server for all its available tools and expose them as if they were local tools. This allows seamless integration with external MCP servers.

For Streamable HTTP MCP endpoints, the proxy performs the MCP init handshake (`initialize` + `notifications/initialized`) and keeps the session id automatically.

Example (single proxy tool):

```yaml
tools:
  - name: my_mcp_tool
    type: mcp_server
    enabled: true
    description: "Proxy to another MCP server"
    server_url: "http://localhost:8080/endpoint"
    forward_args: true       # Optional: forward arguments (default: true)
    timeout: 30              # Optional: timeout in seconds (default: 30)
    auth_username: "${DRUPAL_MCP_USER}"    # Optional: basic auth username
    auth_password: "${DRUPAL_MCP_PASS}"    # Optional: basic auth password
    input_schema:
      type: object
      properties:
        query:
          type: string
      required:
        - query
```

Example (dynamic tool exposure):

```yaml
tools:
  - name: drupal_mcp
    type: mcp_server
    enabled: true
    description: "Proxy to Drupal MCP server"
    server_url: "https://drupal-project.ddev.site/mcp/post"
    auth_username: "${DRUPAL_MCP_USER}"
    auth_password: "${DRUPAL_MCP_PASS}"
    expose_remote_tools: true    # Dynamically fetch and expose remote tools
    tool_prefix: "drupal_"       # Optional: prefix remote tool names
    timeout: 30
```

## Logging

View MCP server logs:

```bash
tail -f /tmp/wdrmcp.log
```
