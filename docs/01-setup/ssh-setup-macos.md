# SSH Setup From macOS

## Connect To Ubuntu Server
```bash
ssh username@server-ip
```

Example:
```bash
ssh rajitha@192.168.10.82
```

## Generate SSH Key
```bash
ssh-keygen -t ed25519
```

## Copy SSH Key To Server
```bash
ssh-copy-id username@server-ip
```
