Absolutely. We’ll make this a **real hands-on AWS Linux Administrator + Troubleshooting project**, not just a list of Linux commands.

## Project: AWS Linux Administration & Troubleshooting Lab

We’ll use **Amazon Linux 2023 on an AWS EC2 instance** and deliberately create problems that you will diagnose and fix.

### Project progression

| Phase | Area                               | Troubleshooting practice                       |
| ----- | ---------------------------------- | ---------------------------------------------- |
| 1     | AWS EC2 + Linux fundamentals       | SSH, hostname, OS, filesystem, processes       |
| 2     | Filesystem & storage               | Disk full, inode full, mount problems          |
| 3     | Users & groups                     | Login failures, expired accounts, group access |
| 4     | Permissions & ACL                  | `chmod`, `chown`, ACL failures                 |
| 5     | Package management                 | Broken/missing packages                        |
| 6     | Services & systemd                 | Failed services, ports, startup failures       |
| 7     | Processes & jobs                   | High CPU, memory, zombie processes             |
| 8     | Networking                         | DNS, routes, ports, connectivity               |
| 9     | SSH troubleshooting                | Authentication, keys, permissions, `sshd`      |
| 10    | Logs & troubleshooting methodology | `journalctl`, `/var/log`, RCA                  |
| 11    | Web server                         | Nginx/Apache failures                          |
| 12    | Storage administration             | EBS, partitions, filesystems, mounts           |
| 13    | Performance troubleshooting        | CPU/RAM/disk/I/O bottlenecks                   |
| 14    | Security hardening                 | SSH, firewall, permissions, sudo               |
| 15    | Bash automation                    | Monitoring and troubleshooting scripts         |
| 16    | Advanced incidents                 | Multi-layer production-style failures          |
| 17    | Final project                      | Complete Linux production incident simulation  |

The important part is that **I won't simply give you the solution immediately**. For troubleshooting labs, I'll give you the symptoms and let you investigate using Linux commands. Then we'll analyze your output together.

---

# Phase 1 — Create the AWS Linux Server

We'll start with a small EC2 instance to keep the lab cost-conscious.

### Target architecture

```text
Your Windows 11 Laptop
        |
        | SSH
        v
+---------------------------+
|       AWS EC2             |
|                           |
|   Amazon Linux 2023       |
|                           |
|   Linux Admin Lab         |
|                           |
|  Users                    |
|  Groups                   |
|  Files                    |
|  Permissions              |
|  Services                 |
|  Networking               |
|  Logs                     |
|  Storage                  |
|  Monitoring               |
+---------------------------+
```

Later we'll add:

```text
                    AWS
                     |
             +-------+-------+
             |               |
          EC2-01           EC2-02
          Linux            Linux
             |               |
             +-------+-------+
                     |
                Troubleshooting
                   Scenarios
```

For now, **one EC2 instance is enough**.

---

# Step 1 — Create EC2

Open AWS Console → **EC2 → Launch Instance**.

Use approximately:

### Name

```text
linux-admin-lab
```

### AMI

```text
Amazon Linux 2023
```

Prefer the current **Amazon Linux 2023 AMI** available in your AWS region.

### Instance type

For this lab:

```text
t3.micro
```

If your account's current free/credit eligibility differs, verify the estimated cost before launching.

### Key pair

Create:

```text
linux-admin-lab-key
```

Download:

```text
linux-admin-lab-key.pem
```

Keep the `.pem` file private.

---

# Step 2 — Security Group

Create:

```text
linux-admin-lab-sg
```

Inbound rule:

| Type | Port | Source |
| ---- | ---: | ------ |
| SSH  |   22 | My IP  |

**Do not use `0.0.0.0/0` for SSH** for this learning server unless there is a specific reason.

Outbound can remain the default.

---

# Step 3 — Storage

For the initial server:

```text
Root volume: 8–10 GB
Type: gp3
```

We will later practice:

* disk usage
* inode usage
* mounting
* filesystem creation
* EBS troubleshooting
* disk expansion

So don't add unnecessary storage yet.

---

# Step 4 — Launch

Launch the instance.

You should eventually see something similar to:

```text
Instance name: linux-admin-lab
State: Running
Public IPv4: xxx.xxx.xxx.xxx
```

---

# Step 5 — Connect from Windows PowerShell

Since you're using Windows + PowerShell, go to the directory containing your key.

For example:

```powershell
cd "C:\Users\<your-user>\Downloads"
```

Check:

```powershell
dir linux-admin-lab-key.pem
```

Then connect:

```powershell
ssh -i .\linux-admin-lab-key.pem ec2-user@<EC2-PUBLIC-IP>
```

Example:

```powershell
ssh -i .\linux-admin-lab-key.pem ec2-user@54.123.45.67
```

You should get something similar to:

```text
       __|  __|_  )
       _|  (     /
      ___|\___|___|

Amazon Linux 2023
https://aws.amazon.com/linux/amazon-linux-2023/

[ec2-user@ip-10-0-1-25 ~]$
```

---

# Step 6 — First Linux Investigation

**Don't install anything yet.**

Run these commands one at a time:

```bash
whoami
```

```bash
hostname
```

```bash
hostnamectl
```

```bash
cat /etc/os-release
```

```bash
uname -r
```

```bash
pwd
```

```bash
ls -la
```

```bash
df -h
```

```bash
free -h
```

```bash
uptime
```

```bash
ip addr
```

Then:

```bash
systemctl --failed
```

---

# Phase 1 Troubleshooting Method

I want you to learn this pattern because it is extremely important for Linux support/DevOps jobs.

When someone says:

> "The Linux server has a problem."

Don't immediately restart things.

Use:

```text
1. Understand
      ↓
2. Observe
      ↓
3. Identify
      ↓
4. Test
      ↓
5. Fix
      ↓
6. Verify
      ↓
7. Document
```

For example:

```text
Application is down
        ↓
Is the server reachable?
        ↓
Is the process running?
        ↓
Is the service running?
        ↓
Is the port listening?
        ↓
Are there errors in logs?
        ↓
Is networking working?
        ↓
Fix
        ↓
Verify
        ↓
RCA
```

This is the mindset we'll develop throughout the project.

---

# Our First Troubleshooting Exercise

After connecting to EC2, run:

```bash
hostname
```

```bash
cat /etc/os-release
```

```bash
uname -r
```

```bash
uptime
```

```bash
df -h
```

```bash
free -h
```

```bash
systemctl --failed
```

Then send me the **complete output**.

### Don't proceed to the next phase yet.

I'll use your actual server output to start **Phase 1: Linux Health Check**, and then we'll begin deliberately breaking/fixing the server.

Our eventual troubleshooting scenarios will include things like:

```text
🔥 "Server is slow"
🔥 "SSH is not working"
🔥 "User cannot log in"
🔥 "Permission denied"
🔥 "Disk is 100% full"
🔥 "Service won't start"
🔥 "Port is not reachable"
🔥 "DNS is not resolving"
🔥 "Website is down"
🔥 "CPU is 100%"
🔥 "Memory is exhausted"
🔥 "Filesystem won't mount"
🔥 "Process is consuming resources"
🔥 "Application works locally but not remotely"
🔥 "Server rebooted unexpectedly"
```

By the final phase, you'll be diagnosing **multi-layer incidents like a Linux/DevOps production support engineer**, rather than memorizing commands.
