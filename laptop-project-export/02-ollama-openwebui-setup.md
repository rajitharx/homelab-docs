# Ollama and Open WebUI Setup

## Goal

Configure Ollama on the homelab server and expose Open WebUI for browser access from Mac and local network.

## What We Did

1. Installed/configured Ollama on the server.
2. Set up Open WebUI using Docker.
3. Exposed Open WebUI on port `3000`.
4. Verified the container status.
5. Troubleshot access issues from Mac.

## Docker Container Observed

```text
ff2ffbd2cc83   ghcr.io/open-webui/open-webui:main   "bash start.sh"   Up   0.0.0.0:3000->8080/tcp   open-webui
```

## Issue Faced

Open WebUI was accessible from the server, but not from Mac using:

```text
http://192.168.10.82:3000/
```

Firewall was checked:

```bash
sudo ufw status
```

Result:

```text
inactive
```

Ping worked. Curl from the server worked:

```bash
curl http://192.168.10.82:3000
```

## Finding

The issue was browser-specific. Safari was able to access the UI.

## Useful Commands

Check container:

```bash
docker ps
```

Check logs:

```bash
docker logs open-webui
```

Restart container:

```bash
docker restart open-webui
```

Run Open WebUI:

```bash
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  ghcr.io/open-webui/open-webui:main
```

## Lessons Learned

- `0.0.0.0:3000->8080/tcp` means Docker is exposing port 3000 to the network.
- If server curl works and firewall is inactive, test with another browser.
- Safari worked when another browser did not.
- Docker health status may show `starting` or `unhealthy` during initial startup. Check logs before assuming failure.

## Suggested Repo Location

```text
homelab-docs/docs/setup/ollama-openwebui.md
```
