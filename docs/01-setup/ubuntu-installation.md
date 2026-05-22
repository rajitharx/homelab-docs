# Ubuntu Server Installation

## Create Bootable USB
- Download Ubuntu Server 24.04 LTS
- Use Balena Etcher or Rufus
- Flash ISO into USB drive

## Boot Installation
- Press F12 during startup
- Select USB boot device

## Recommended Setup
- Use entire disk
- Install OpenSSH Server
- Create normal user account

## First Update
```bash
sudo apt update
sudo apt upgrade -y
```
