````md
# Homelab AI Server Setup Journey (Ollama + Open WebUI)

## Introduction

The goal of this setup was to transform a personal Ubuntu homelab server into a centralized AI server capable of serving local LLMs to multiple devices such as:
- Personal MacBook
- Office Lenovo laptop
- Future mobile/tablet access

The target architecture was:

```text
MacBook
    \
Office Laptop ---> Homelab Ubuntu Server (Ollama + Open WebUI)
    /
```

The idea was simple:
- Run the AI models once on the server
- Access them remotely from any device inside the home network

---

# Phase 1 — Installing Ubuntu Server

Ubuntu Server was installed successfully on the homelab machine.

After installation:
- SSH was enabled
- GUI was later installed for convenience
- Network connectivity was verified

---

# Phase 2 — Installing Ollama

Ollama was selected because:
- Lightweight
- Easy local LLM management
- Native Linux support
- Strong compatibility with coding models

Installation command:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Verify service:

```bash
systemctl status ollama
```

---

# Phase 3 — Downloading the First AI Model

Initial test model:

```bash
ollama pull llama3.2:3b
```

Later, coding-focused models were recommended:

```bash
ollama pull qwen2.5-coder:7b
```

Testing the model locally:

```bash
ollama run llama3.2:3b
```

At this stage, Ollama was working only inside the server itself.

---

# Phase 4 — Problem: Ollama Not Accessible from MacBook

Initial network check:

```bash
ss -tulnp | grep 11434
```

Output:

```text
127.0.0.1:11434
```

Problem:
- Ollama was listening only on localhost
- External devices could not connect

---

# Phase 5 — Exposing Ollama to the Network

We modified the Ollama systemd override configuration:

```bash
sudo systemctl edit ollama
```

Added:

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```

Reloaded services:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

Verification:

```bash
ss -tulnp | grep 11434
```

Successful result:

```text
*:11434
```

Now Ollama was listening on all network interfaces.

---

# Phase 6 — Testing Remote Access

Server IP was identified:

```bash
hostname -I
```

Example:

```text
192.168.10.82
```

From MacBook:

```bash
curl http://192.168.10.82:11434/api/tags
```

Result:
- Success
- Ollama API accessible remotely

At this point, the server officially became a network AI server.

---

# Phase 7 — Installing Open WebUI

To provide a ChatGPT-style interface, Open WebUI was installed.

Docker was required.

Install Docker:

```bash
sudo apt update
sudo apt install -y docker.io
```

Enable Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

---

# Phase 8 — Problem: Docker Permission Issues

Attempting:

```bash
newgrp docker
```

Result:

```text
permission denied
```

Cause:
- Current shell session not refreshed
- Docker group membership not applied yet

Temporary workaround:
- Use `sudo docker`

Permanent fix:
- Re-login or reboot later

---

# Phase 9 — Deploying Open WebUI

Initial deployment:

```bash
sudo docker run -d \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

---

# Phase 10 — Problem: Open WebUI Unhealthy

Docker status showed:

```text
(unhealthy)
```

Logs investigation:

```bash
sudo docker logs open-webui --tail 100
```

Cause:
- Open WebUI unable to communicate properly with Ollama

Fix:
- Recreate container with explicit Ollama endpoint

```bash
sudo docker rm -f open-webui
```

Recreated container:

```bash
sudo docker run -d \
  -p 3000:8080 \
  -e OLLAMA_BASE_URL=http://192.168.10.82:11434 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

Result:
- Open WebUI became operational

---

# Phase 11 — Problem: Mac Browser Could Not Access WebUI

Symptoms:
- `curl http://192.168.10.82:3000` worked
- Ping worked
- Safari worked
- Chrome failed

Investigation included:
- Firewall checks
- Network isolation checks
- Port verification
- Docker verification

Firewall status:

```bash
sudo ufw status
```

Result:

```text
inactive
```

Conclusion:
- Not a server issue
- Chrome/proxy/security/VPN related issue

Final workaround:
- Use Safari successfully

---

# Final Working Setup

## Server Components

| Component | Status |
|---|---|
| Ubuntu Server | ✅ |
| Ollama | ✅ |
| Qwen Coder Model | ✅ |
| Docker | ✅ |
| Open WebUI | ✅ |
| Network Access | ✅ |

---

# Final URLs

## Ollama API

```text
http://192.168.10.82:11434
```

## Open WebUI

```text
http://192.168.10.82:3000
```

---

# Final Result

The homelab server successfully became a private AI server capable of:
- Running local LLMs
- Serving multiple devices
- Providing ChatGPT-style UI
- Supporting coding-focused AI workflows
- Enabling future integrations with VS Code, Continue.dev, and AI agents

The system is now ready for:
- Local AI coding assistant
- AI experimentation
- Document RAG
- Multi-device AI access
- Future automation workflows

---

# Recommended Future Enhancements

## AI Models

```bash
ollama pull qwen2.5-coder:14b
ollama pull mistral:7b
```

## VS Code Integration

Recommended tools:
- Continue.dev
- Cline

## Remote Secure Access

Future improvements:
- Tailscale
- WireGuard VPN

## Persistent Storage

Future additions:
- NAS storage
- Backup strategy
- GPU acceleration

---

# Lessons Learned

1. Ollama listens only on localhost by default
2. Docker permissions require re-login
3. Open WebUI may require explicit Ollama endpoint configuration
4. Browser issues are sometimes unrelated to server/network setup
5. Curl testing is extremely valuable for isolating issues
6. Homelab AI setup is surprisingly achievable with lightweight tooling

---

# Outcome

A fully functional private AI homelab environment was successfully built using:
- Ubuntu Server
- Ollama
- Docker
- Open WebUI

The environment now acts as a centralized AI platform for development and experimentation.
````
