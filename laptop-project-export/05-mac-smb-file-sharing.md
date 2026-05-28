# Mac SMB File Sharing with Homelab Server

## Goal

Share a folder from Mac to the homelab server for easy file transfers.

## Mac Folder

```text
/Users/rajitha/HomelabShare
```

## Issues Faced

Mount command failed with:

```text
mount error(13): Permission denied
```

Also tested with `smbclient`, but it was not installed initially:

```bash
sudo apt install smbclient
```

Then login failed with:

```text
session setup failed: NT_STATUS_LOGON_FAILURE
```

## Important Clarification

The SMB username/password should be the Mac user account credentials that have permission to access the shared folder.

## Check Mac Username

On Mac:

```bash
whoami
```

## Enable File Sharing on Mac

1. System Settings
2. General
3. Sharing
4. Turn on File Sharing
5. Add `/Users/rajitha/HomelabShare` as shared folder
6. Click info/options
7. Enable SMB sharing for the Mac user
8. Ensure the user has permission for the shared folder

## Test from Ubuntu Server

Install SMB client:

```bash
sudo apt update
sudo apt install smbclient cifs-utils
```

List shares:

```bash
smbclient -L //<MAC_IP> -U <mac_username>
```

Mount share:

```bash
sudo mkdir -p /mnt/macshare

sudo mount -t cifs //<MAC_IP>/HomelabShare /mnt/macshare \
  -o username=<mac_username>,vers=3.0,uid=$(id -u),gid=$(id -g)
```

## Lessons Learned

- Use Mac login credentials, not Ubuntu credentials.
- Mac SMB sharing must be enabled for the user.
- Folder permission and SMB permission are both important.
- `NT_STATUS_LOGON_FAILURE` usually means wrong username/password or SMB not enabled for that Mac user.

## Suggested Repo Location

```text
homelab-docs/docs/file-sharing/mac-smb-share.md
```
