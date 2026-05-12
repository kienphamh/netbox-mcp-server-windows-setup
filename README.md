# NetBox MCP Server — Windows Setup Guide

A step-by-step guide to setting up the [NetBox MCP Server](https://github.com/netboxlabs/netbox-mcp-server) on Windows, with integration for both **VSCode (GitHub Copilot)** and **Claude Desktop**.

This guide documents real issue encountered during setup on Windows, including PATH problems, config formatting, and API token errors.

---

## Prerequisites

Before starting, make sure you have the following installed:

- [Git for Windows](https://git-scm.com/download/win)
- [uv](https://astral.sh/uv) (Python package manager)
- [VSCode](https://code.visualstudio.com/) with GitHub Copilot **or** [Claude Desktop](https://claude.ai/download)
- A NetBox instance or access to [demo.netbox.dev](https://demo.netbox.dev)

### Install uv (if not already installed)

Open Git Bash and run:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Then restart your terminal.

---

## Step 1: Clone the NetBox MCP Server

```bash
git clone https://github.com/netboxlabs/netbox-mcp-server.git
cd netbox-mcp-server
uv sync
```

---

## Step 2: Get a NetBox API Token

1. Go to [demo.netbox.dev](https://demo.netbox.dev) and log in
2. Click your username (top right) → **API Tokens** → **Add a Token**. Any API token version is sufficient for this demo
3. Copy the full token immediately, since it will only be shown once

> **Tip:** Use the `admin` privilege account for testing. Regular user accounts may lack permissions and cause `403 Forbidden` errors.

> **Note:** The demo site resets periodically, so your token will expire. Generate a new one if you get 403 errors.

---

## Step 3: Find your uv path

VSCode and Claude Desktop need the **full path** to `uv.exe` — they can't rely on your terminal's PATH.

Run this in Git Bash:

```bash
which uv
```

You'll get something like:

```
/c/Users/YourName/.local/bin/uv
```

Convert it to a Windows path by:
- Replacing `/c/` with `C:\\`
- Replacing `/` with `\\`
- Adding `.exe` at the end

**Example:**
```
/c/Users/YourName/.local/bin/uv
→ C:\\Users\\YourName\\.local\\bin\\uv.exe
```

---

## Setup for VSCode (GitHub Copilot)

### 1. Open the MCP config file

Press `Ctrl + Shift + P` → type **MCP: Open User Configuration** → press Enter

### 2. Paste the following config

```json
{
    "servers": {
        "netbox": {
            "command": "C:\\Users\\YourName\\.local\\bin\\uv.exe",
            "args": [
                "--directory",
                "C:\\Users\\YourName\\path\\to\\netbox-mcp-server",
                "run",
                "netbox-mcp-server"
            ],
            "env": {
                "NETBOX_URL": "https://demo.netbox.dev/",
                "NETBOX_TOKEN": "your_API_token"
            }
        }
    }
}
```

Replace `YourName`, the directory path, and `your_API_token` with your actual values.

### 3. Start the server

Press `Ctrl + Shift + P` → **MCP: List Servers** → click **netbox** → **Start**

### 4. Verify it's running

You should see in the Output panel (`View → Output → MCP: netbox`):

```
[info] Connection state: Running
[info] Discovered 4 tools
```

### 5. Test it in GitHub Copilot
First, ensure netbox and pylance mcp server is checked in the configure tools of the agent. Open Copilot Chat (`Ctrl + Alt + I`), switch to **Agent** mode, and try:

```
Using netbox mcp server. List all devices in NetBox.
```

---

## Setup for Claude Desktop

### 1. Open the config file

In Claude Desktop, go to **Settings → Developer → Developer**, it'll lead you to `claude_desktop_config.json`

### 2. Paste the following config

```json
"mcpServers": {
        "netbox": {
            "command": "C:\\Users\\YourName\\.local\\bin\\uv.exe",
            "args": [
                "--directory",
                "C:\\Users\\YourName\\path\\to\\netbox-mcp-server",
                "run",
                "netbox-mcp-server"
            ],
            "env": {
                "NETBOX_URL": "https://demo.netbox.dev/",
                "NETBOX_TOKEN": "your_API_token"
            }
        }
    }
```

### 3. Restart Claude Desktop

Fully close and reopen Claude Desktop. The NetBox tools will appear automatically when the server connects.

---

## Common Errors & Fixes

### `Error spawn uv ENOENT`
VSCode can't find `uv` because it doesn't use your terminal's PATH. Fix: use the **full path** to `uv.exe` in the `command` field (see Step 3).

### `403 Forbidden`
Your API token has expired (the demo site resets periodically) or the user account doesn't have the right permissions. Fix: generate a new token using the `admin` account.

### Server keeps showing `Stopping server`
This is normal when you manually click restart, the server stops first, then starts. Check **MCP: List Servers** for the actual status (`running`, `stopped`, or `error`).

### Config changes not saving
If VSCode keeps using the old config, edit the file directly at:
```
C:\Users\YourName\AppData\Roaming\Code\User\mcp.json
```

---

## Testing the Integration

Once connected, try these prompts in Copilot or Claude:

- `List all sites in NetBox`
- `Show me all devices in NetBox`
- `List all IP addresses in NetBox`
- `Show me all VLANs in NetBox`
- `Which devices are in which sites?`

---

## Notes

- The `[server stderr]` warnings in VSCode's output are **normal** — they are just server logs, not errors
- The NetBox demo site resets daily, so tokens and data are temporary
- Always use double backslashes `\\` in Windows paths inside JSON

---

## Resources

- [netboxlabs/netbox-mcp-server](https://github.com/netboxlabs/netbox-mcp-server)
- [NetBox Demo](https://demo.netbox.dev)
- [uv documentation](https://docs.astral.sh/uv)
- [MCP documentation](https://modelcontextprotocol.io)
