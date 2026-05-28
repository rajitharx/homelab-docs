# Local MCP Server Setup

## Goal

Create and connect a local engineering MCP server for use from VS Code / Copilot-style tooling.

## What We Tried

Started the MCP server using Node:

```bash
node mcp-server/server.js
```

At one point, Node showed an ES module warning:

```text
Warning: Failed to load the ES module: server.js.
Make sure to set "type": "module" in the nearest package.json file or use the .mjs extension.
```

## Meaning

Node was treating the file as CommonJS, but the code was likely written as ES module syntax.

## Options to Fix

### Option 1: Add type module

In `package.json`:

```json
{
  "type": "module"
}
```

### Option 2: Rename file

```text
server.js -> server.mjs
```

## MCP Config Attempt

Initial config:

```json
{
  "servers": {
    "local-engineering-mcp": {
      "command": "node",
      "args": [
        "/Users/rajitha/Development/DailyDashboard/mcp-server/server.js"
      ]
    }
  }
}
```

Later changed to remote MCP bridge config:

```json
{
  "servers": {
    "local-engineering-mcp": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "http://localhost:5050/mcp",
        "--allow-http"
      ]
    }
  }
}
```

## Server Health Check

Checked MCP endpoint:

```bash
curl http://192.168.10.82:5050/mcp
```

Health response:

```text
Local Engineering MCP server health: running.
```

## VS Code Logs Observed

```text
Starting server local-engineering-mcp
Connection state: Starting
Connection state: Running
Using automatically selected callback port
Discovering OAuth server configuration...
```

## Key Lesson

For a remote HTTP MCP endpoint, use `mcp-remote` as the bridge from VS Code/Copilot to the HTTP MCP server.

## Suggested Repo Location

```text
homelab-docs/docs/mcp/local-engineering-mcp.md
```
