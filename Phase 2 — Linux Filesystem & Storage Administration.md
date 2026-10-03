Great. Your Phase 1 health check is clean.

### Phase 1 findings

| Check          | Result                            |
| -------------- | --------------------------------- |
| OS             | Amazon Linux 2023                 |
| Kernel         | `6.18.51-120.163.amzn2023.x86_64` |
| Root disk      | 8 GB                              |
| Disk used      | 20%                               |
| Available disk | 6.4 GB                            |
| RAM            | 7.6 GiB                           |
| RAM used       | ~176 MiB                          |
| Swap           | None                              |
| Uptime         | 55 minutes                        |
| Load average   | `0.01, 0.01, 0.00`                |

Nothing currently indicates a disk, memory, or CPU problem.

# Phase 2 — Linux Filesystem & Storage Administration

Now we're going to learn one of the **most important Linux troubleshooting areas**:

```text
Filesystem
   ↓
Disk space
   ↓
Inodes
   ↓
Mount points
   ↓
Permissions
   ↓
Files
   ↓
Troubleshooting
```

We'll first investigate your existing filesystem **without changing anything**.

## 2.1 — Filesystem structure

Run:

```bash
ls /
```

Then:

```bash
ls -lah /
```

Then:

```bash
du -sh /*
```

If `du` reports some `/proc`, `/sys`, or `/run` entries changing or inaccessible, don't worry yet.

---

## 2.2 — Understand your disk

Run:

```bash
lsblk
```

Then:

```bash
lsblk -f
```

Then:

```bash
sudo fdisk -l
```

You should see something related to:

```text
nvme0n1
├─nvme0n1p1
└─nvme0n1p128
```

We want you to understand the relationship:

```text
EBS Volume
    ↓
NVMe device
    ↓
Partition
    ↓
Filesystem
    ↓
Mount point
```

In your case, your root filesystem is currently:

```text
/dev/nvme0n1p1
        ↓
       /
```

---

# 2.3 — Check filesystem type

Run:

```bash
df -Th
```

Pay particular attention to:

```text
Filesystem
Type
Size
Used
Avail
Use%
Mounted on
```

You'll likely see `xfs` for the root filesystem on Amazon Linux 2023.

---

# 2.4 — Learn inode troubleshooting

Disk space isn't the only way a filesystem can become full.

Check:

```bash
df -i
```

Compare:

```text
df -h
```

with:

```text
df -i
```

Think of it this way:

```text
Disk capacity
     |
     +---- blocks → df -h
     |
     +---- inodes → df -i
```

A filesystem can have available GB but still fail to create files if it runs out of inodes.

---

# 2.5 — Investigate `/var`

`/var` is particularly important for Linux administrators because it commonly contains:

```text
/var/log
/var/cache
/var/tmp
/var/lib
```

Run:

```bash
sudo du -sh /var/*
```

Then:

```bash
sudo du -sh /var/log/*
```

Don't delete anything yet.

---

# 2.6 — Check mounted filesystems

Run:

```bash
mount | column -t
```

And:

```bash
findmnt
```

You don't need to memorize every mount.

The important troubleshooting question is:

> **Which device is mounted where?**

For example:

```text
/dev/nvme0n1p1 → /
```

---

# Your first real troubleshooting scenario

After collecting the information above, we're going to simulate:

> **INCIDENT #001 — "The Linux server says there is no space left on the device."**

But **don't create the failure yet**.

First run these commands:

```bash
ls /
```

```bash
ls -lah /
```

```bash
du -sh /*
```

```bash
lsblk
```

```bash
lsblk -f
```

```bash
df -Th
```

```bash
df -i
```

```bash
sudo du -sh /var/*
```

```bash
sudo du -sh /var/log/*
```

```bash
findmnt
```

### Send me the outputs.

Then I'll give you the **Incident #001 symptoms**, and you'll troubleshoot it yourself using commands rather than me giving you the answer directly.
