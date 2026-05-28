# Homelab Storage and Disk Resize Notes

## Goal

Understand current disk layout and how to deallocate/reduce around 60GB if needed.

## Context

The server showed multiple loop devices from Snap packages and the main disk layout was inspected using commands like:

```bash
lsblk
```

## Useful Commands

Check disks:

```bash
lsblk
```

Check filesystem usage:

```bash
df -h
```

Check partitions:

```bash
sudo fdisk -l
```

Check LVM, if used:

```bash
sudo pvs
sudo vgs
sudo lvs
```

## Important Warning

Reducing disk or partition size is risky. Always back up before resizing.

## General Rule

- Expanding is usually safer.
- Shrinking needs filesystem resize first, then partition/LV resize.
- If using LVM, reduce filesystem before reducing logical volume.

## Suggested Repo Location

```text
homelab-docs/docs/setup/storage-disk-resize.md
```
