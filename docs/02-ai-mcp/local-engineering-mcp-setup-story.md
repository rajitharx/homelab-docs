# Building a Generic Local MCP Server on a Homelab (Ubuntu + Docker + VS Code)

## Overview

This document captures the full journey of setting up a generic MCP (Model Context Protocol) server using:

* Ubuntu Homelab Server
* Docker
* .NET 8
* Official MCP C# SDK
* VS Code
* GitHub Copilot Agent Mode
* MacBook as the client machine

The goal was to create a reusable and generic engineering MCP server that could later be mapped to projects and tools.

---

# Initial Understanding

At first, there was confusion between:

* LLM
* MCP

We clarified:

| Component | Purpose              |
| --------- | -------------------- |
| LLM       | The AI brain         |
| MCP       | Tool connector layer |

The local setup already had:

* Ollama
* Open WebUI
* Ubuntu server

So the missing piece was:

* MCP server

The decision was made to:

* keep the MCP generic first
* avoid tying it to any office/internal project
* later map projects dynamically

---

# Architecture Goal

Final desired architecture:

```text
MacBook (VS Code)
        ↓
GitHub Copilot Agent
        ↓
MCP Client
        ↓
Ubuntu Homelab MCP Server
        ↓
Generic Engineering Tools
```

---

# Creating the MCP Server

## Step 1 — Install .NET 8 SDK

On Ubuntu:

```bash
sudo apt install -y dotnet-sdk-8.0
```

Verified:

```bash
dotnet --version
```

---

# Step 2 — Create MCP Project

Created project:

```bash
mkdir -p ~/homelab-mcp
cd ~/homelab-mcp

dotnet new console -n LocalEngineering.McpServer
```

Installed packages:

```bash
dotnet add package ModelContextProtocol
dotnet add package Microsoft.Extensions.Hosting
```

---

# First Issue — Static Class Error

Initial implementation used:

```csharp
public static class SystemTools
```

Build failed with:

```text
static types cannot be used as type arguments
```

## Root Cause

`.WithTools<T>()` expects a normal class.

## Fix

Changed to:

```csharp
public class SystemTools
```

Problem solved.

---

# MCP Connection Attempt Using SSH

Initial attempt used:

```json
{
  "servers": {
    "local-engineering-mcp": {
      "command": "ssh",
      "args": [
        "homelab@[server-ip]",
        "cd /home/homelab/homelab-mcp/LocalEngineering.McpServer && dotnet run"
      ]
    }
  }
}
```

---

# Second Issue — SSH "No route to host"

VS Code logs:

```text
ssh: connect to host [server-ip] port 22: No route to host
```

However:

* direct SSH from Mac terminal worked

## Root Cause

VS Code MCP process networking behaved differently from terminal SSH execution.

## Decision

Instead of fighting SSH stdio transport:

* migrated MCP server to HTTP transport

---

# Converting MCP to HTTP Server

Installed:

```bash
dotnet add package ModelContextProtocol.AspNetCore
```

Changed project SDK:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

Converted MCP transport from:

```csharp
.WithStdioServerTransport()
```

to:

```csharp
.WithHttpTransport()
```

Mapped endpoint:

```csharp
app.MapMcp("/mcp");
```

Server now listened on:

```text
http://0.0.0.0:5050
```

---

# Third Issue — HTTP Reachability

Mac initially failed:

```text
curl: Failed to connect
```

## Root Cause

Server not listening properly or container not running.

Eventually confirmed:

```bash
curl http://[server-ip]:5050/mcp
```

returned:

```json
{
  "error": {
    "message": "Mcp-Session-Id header is required"
  }
}
```

This was actually GOOD.

It confirmed:

* network worked
* MCP endpoint was alive

---

# Fourth Issue — VS Code "fetch failed"

VS Code continued showing:

```text
fetch failed
```

## Root Cause

Mismatch between:

* VS Code MCP transport expectations
* MCP session handling

## Fix Attempt

Enabled stateless mode:

```csharp
options.Stateless = true;
```

Still unstable.

---

# Decision — Use `mcp-remote`

Instead of direct HTTP connection from VS Code:

* introduced `mcp-remote`

Architecture became:

```text
VS Code
  ↓
mcp-remote
  ↓
HTTP MCP Server
```

---

# Fifth Issue — Node.js Version Too Old

Logs showed:

```text
Unsupported engine
required: node >= 20.18.1
current: node v19.6.0
```

## Fix

Updated Node.js on Mac.

After upgrade:

* `mcp-remote` executed correctly

---

# Sixth Issue — HTTP Safety Restriction

Error:

```text
Non-HTTPS URLs are only allowed for localhost
```

## Fix

Added:

```json
"--allow-http"
```

to `mcp-remote`.

---

# Seventh Issue — EHOSTUNREACH

Even though browser access worked:

```bash
curl http://[server-ip]:5050/mcp
```

VS Code still failed with:

```text
EHOSTUNREACH
```

## Root Cause

Node.js runtime inside VS Code could not properly route to LAN IP.

---

# Final Working Solution — SSH Tunnel

Created SSH tunnel:

```bash
ssh -L 5050:localhost:5050 homelab@[server-ip]
```

This mapped:

```text
Mac localhost:5050
        ↓
Ubuntu localhost:5050
```

Then updated VS Code MCP config:

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

---

# Success

VS Code finally connected successfully.

Validation:

```text
Local Engineering MCP server health: running.
```

MCP tools were now accessible from:

* GitHub Copilot Agent Mode
* VS Code

---

# Dockerization

Instead of running:

```bash
dotnet run
```

manually forever, Docker was introduced.

## Dockerfile

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app

COPY --from=build /app/publish .
EXPOSE 5050

ENTRYPOINT ["dotnet", "LocalEngineering.McpServer.dll"]
```

Build:

```bash
docker build -t local-engineering-mcp .
```

Run:

```bash
docker run -d \
  --name local-engineering-mcp \
  --restart unless-stopped \
  -p 5050:5050 \
  local-engineering-mcp
```

---

# Final Working Architecture

```text
MacBook
 ├── VS Code
 ├── GitHub Copilot Agent
 ├── mcp-remote
 └── SSH Tunnel
          ↓
Ubuntu Homelab
 ├── Docker
 ├── LocalEngineering.McpServer
 └── MCP HTTP Endpoint
```

---

# Lessons Learned

## 1. MCP is not the AI

The LLM provides reasoning.

MCP provides:

* tools
* integrations
* external capabilities

---

## 2. HTTP MCP is easier for homelabs

Compared to SSH stdio transport:

* easier debugging
* easier Dockerization
* easier scaling

---

## 3. VS Code MCP ecosystem is still evolving

Many issues encountered were due to:

* transport compatibility
* Node.js requirements
* HTTP restrictions
* session expectations

---

## 4. SSH Tunnel solved routing instability

Using:

```bash
ssh -L
```

was the most stable approach.

---

# Next Steps

Potential future tools:

* File system access
* Git status
* Read file
* Safe terminal execution
* `dotnet build`
* `dotnet test`
* SQL analysis
* Log inspection

The MCP server is now generic and ready for future project integrations.
