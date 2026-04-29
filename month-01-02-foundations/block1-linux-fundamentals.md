# Block 1 — Linux Terminal Fundamentals

**Date:** April 26, 2026  
**Environment:** WSL Ubuntu on Windows (VS Code integrated terminal)  
**Roadmap:** Cloud Skills Roadmap — Month 1–2 Foundations  
**Time spent:** ~2 hours  

---

## What I Learned

Linux is the foundation for Docker, Vagrant, SSH, Kubernetes, and cloud VMs. Every tool in the DevOps/Cloud stack runs on Linux. This block covers the terminal skills needed before touching any project.

---

## Environment Setup

| Detail | Value |
|---|---|
| OS | Windows with WSL2 |
| Linux distro | Ubuntu (WSL) |
| Terminal | VS Code integrated terminal |
| Python version | 3.12.3 |

**Note:** WSL (Windows Subsystem for Linux) provides a native Linux environment on Windows without needing a separate VM. For this block it works better than Vagrant — faster, integrated with VS Code, no overhead.

---

## Commands Practiced

### Navigation

```bash
# Print current working directory
pwd
# Output: /home/rushimaddi

# List all files including hidden, with permissions
ls -la

# Go to home directory
cd ~

# Create a new directory
mkdir cloudskills

# Navigate into it
cd cloudskills

# Go back up one level
cd ..
```

**Key concept:** The Linux filesystem starts at `/` (root). Your home directory is `/home/username`. `~` is a shortcut for your home directory.

---

### File Operations

```bash
# Create an empty file
touch file.txt

# Open and edit in nano text editor
nano file.txt
# Ctrl+O to save, Enter to confirm, Ctrl+X to exit

# Print file contents to terminal
cat file.txt
```

**Key concept:** `nano` is a beginner-friendly terminal text editor. `vim` is more powerful but has a steeper learning curve. For now, `nano` is enough.

---

### Permissions

```bash
# View permissions on all files
ls -la
# Example output: -rw-r--r-- 1 rushimaddi rushimaddi 45 Apr 26 08:30 file.txt
```

**Reading permissions — rwx breakdown:**

```
-  rw-  r--  r--
|   |    |    |
|   |    |    └── Others: read only
|   |    └─────── Group: read only  
|   └──────────── Owner: read + write
└──────────────── File type (- = file, d = directory)
```

```bash
# Change permissions — 755 means owner=rwx, group=rx, others=rx
chmod 755 file.txt

# Run as superuser (admin equivalent)
sudo apt update
```

**chmod number reference:**

| Number | Permission |
|---|---|
| 7 | rwx (read + write + execute) |
| 6 | rw- (read + write) |
| 5 | r-x (read + execute) |
| 4 | r-- (read only) |
| 0 | --- (no permissions) |

---

### Processes

```bash
# Show all running processes
ps aux

# Interactive process viewer — press q to quit
top

# Check service status (works on real Linux servers, not WSL)
systemctl status ssh
```

**WSL note:** `systemctl` doesn't work fully in WSL because systemd isn't enabled by default. On a real Azure VM or Vagrant VM, `systemctl status ssh` shows the SSH service running. This will be demonstrated in Block 5 when connecting to an Azure VM.

---

### Package Management

```bash
# Update package lists
sudo apt update

# Upgrade installed packages
sudo apt upgrade -y

# Install a package
sudo apt install curl -y

# Verify installation
curl --version
```

**Key concept:** `apt` is Ubuntu's package manager — equivalent to Windows Store but for Linux software. Always run `apt update` before installing anything to get the latest package list.

---

## Key Concepts Understood

**Linux filesystem hierarchy:**
```
/               ← root (top of everything)
├── home/       ← user home directories
│   └── rushimaddi/   ← your home = ~
├── etc/        ← system config files
├── var/        ← logs, variable data
├── usr/        ← installed programs
└── tmp/        ← temporary files
```

**sudo:** Stands for "superuser do." Runs a command as administrator. Required for installing packages, changing system files, starting services.

**File permissions matter for Docker:** When containers try to write files or access directories, permission errors are common. Understanding `chmod` and `chown` helps debug these issues.

---

## WSL vs Real Linux Server — Key Differences

| Feature | WSL | Azure VM / Vagrant VM |
|---|---|---|
| systemd / systemctl | Limited | ✅ Full support |
| SSH server | Not running by default | ✅ Running |
| Docker | Needs Docker Desktop | Install directly |
| Performance | Near-native | Native |
| Use case | Daily development | Server simulation, production |

---

## What's Next

**Block 2 — SSH Key Management**  
Generate SSH key pair and understand how passwordless authentication works. This key will be used to SSH into the Azure VM in Block 5.

---

## Resources Used

- [Ryan's Linux Tutorial](https://ryanstutorials.net/linuxtutorial/) — sections 1–6
- [WSL Documentation](https://learn.microsoft.com/en-us/windows/wsl/)

---

*Part of the [Cloud Skills Roadmap](https://github.com/Mrushi1903/cloud-skills-roadmap) — Month 1–2 Foundations*
