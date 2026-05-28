# Recommended Repo Structure for Homelab Docs

## Goal

Keep homelab setup notes, troubleshooting stories, commands, and architecture decisions in a clean GitHub repository.

## Suggested Structure

```text
homelab-docs/
├── README.md
├── docs/
│   ├── setup/
│   │   ├── ubuntu-installation.md
│   │   ├── ollama-openwebui.md
│   │   └── storage-disk-resize.md
│   ├── monitoring/
│   │   └── grafana-prometheus.md
│   ├── mcp/
│   │   └── local-engineering-mcp.md
│   ├── file-sharing/
│   │   └── mac-smb-share.md
│   ├── ai/
│   │   └── local-llm-options.md
│   └── troubleshooting/
│       └── network-and-browser-access.md
├── scripts/
│   └── README.md
└── assets/
    └── screenshots/
```

## Naming Style

Use lowercase kebab-case:

```text
ollama-openwebui.md
grafana-prometheus.md
local-engineering-mcp.md
mac-smb-share.md
```

## Documentation Style

Each doc should include:

```text
# Title

## Goal
## Environment
## What We Did
## Issues Faced
## Fix
## Useful Commands
## Lessons Learned
```

## Why This Structure Works

- Easy to browse in GitHub.
- Easy to search later.
- Commands are grouped by topic.
- Troubleshooting history is preserved.
- Can grow into a proper homelab knowledge base.
