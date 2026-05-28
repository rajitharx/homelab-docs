# Ubuntu Installation and Server Setup

## Goal

Install Ubuntu on the laptop/server and prepare it for homelab usage.

## Key Flow

1. Flash drive was prepared with Ubuntu.
2. During installation, network configuration showed:
   - `enp43s0` - wired Ethernet interface.
   - `wlp0s20f3` - Wi-Fi interface.
3. User chose disk installation options.
4. Server name was discussed.
5. OpenSSH installation was recommended so the server can be accessed remotely.
6. Installation got stuck around third-party drivers.
7. After installation, checked IP address.
8. Discussed whether to install GUI.
9. Installed GUI using Ubuntu Desktop.

## Useful Commands

Check IP address:

```bash
ip addr
hostname -I
```

Install Ubuntu Desktop GUI:

```bash
sudo apt update
sudo apt install ubuntu-desktop
sudo reboot
```

Install OpenSSH server:

```bash
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

## Notes

Installing `ubuntu-desktop` adds a full desktop environment on top of Ubuntu Server. The command line remains fully usable after installing the GUI.

## Lessons Learned

- Install OpenSSH during setup if the machine will be managed remotely.
- GUI is optional, but useful for beginners or quick visual administration.
- CLI remains available even after installing GUI.
