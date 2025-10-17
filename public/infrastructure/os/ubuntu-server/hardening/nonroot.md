---
title: Least Privilege & Non-Root Operation
description: Hardening a new Image to use Least Privilege & Non-Root 
published: true
date: 2025-10-17T22:02:13.052Z
tags: infrastructure, public, os, ubuntu-server, hardening, nonroot
editor: markdown
dateCreated: 2025-10-17T21:52:54.008Z
---

# Least Privilege & Non-Root Operation
## Purpose

Run everything with the minimum permissions required. Admin logins use a normal user + sudo. Services run as dedicated service users (or a constrained deploy user) instead of root. This limits blast radius and aligns with Kubernetes/DevSecOps best practices.

## 1) SSH/Login Accounts (humans)

Create a normal admin user for shell access; use SSH keys and sudo when needed.
> Note: If built through Raspberry PI Imager, this step is already done. Skip to 2
### Create admin user (interactive; sets home & shell)
```bash
sudo adduser dev
```
### Allow sudo (admin group name may be 'sudo' on Ubuntu)
```bash
sudo usermod -aG sudo dev
```

### Copy your SSH key (from your workstation)
```bash
ssh-copy-id dev@<node-ip>
# or paste into: /home/dev/.ssh/authorized_keys (chmod 700 ~/.ssh; chmod 600 authorized_keys)

# Optional: prevent password logins entirely (handled in SSH Hardening)
# /etc/ssh/sshd_config -> PasswordAuthentication no
```

Why: Daily work happens as dev (non-root). Elevation is explicit (sudo) and audited.

## 2) Service Users (daemons)

Each long-running service gets its own user & group with no login shell and minimal filesystem access.

### Option A — system account (recommended)
#### Create a system user, no password, no login shell
```bash
sudo adduser --system --home /var/lib/myapp --group --shell /usr/sbin/nologin myapp
```

#### App directories with tight ownership
```bash
sudo mkdir -p /opt/myapp/bin /var/lib/myapp /var/log/myapp
sudo chown -R myapp:myapp /opt/myapp /var/lib/myapp /var/log/myapp
sudo chmod 750 /opt/myapp /var/lib/myapp
sudo chmod 750 /var/log/myapp
```
#### Option B — run under a shared “deploy” user

If you prefer one deploy user that owns binaries and restarts services:
```bash
sudo adduser deploy
sudo usermod -aG sudo deploy   # (optional; see constrained sudo below)
```

Use constrained sudo so deploy can manage only specific services, not the whole box.

## 3) Run Services as Non-Root (systemd)

Example systemd unit for a Go web service running on port 8080:

### /etc/systemd/system/myapp.service
```bash
[Unit]
Description=MyApp (non-root)
After=network-online.target
Wants=network-online.target

[Service]
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/myapp --config /opt/myapp/config.yaml
Restart=on-failure
RestartSec=3

# Defense-in-depth
NoNewPrivileges=true
ProtectSystem=full
ProtectHome=true
PrivateTmp=true
ProtectHostname=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true
LockPersonality=true
MemoryDenyWriteExecute=true
ReadWritePaths=/var/lib/myapp /var/log/myapp
AmbientCapabilities=
CapabilityBoundingSet=

[Install]
WantedBy=multi-user.target
```

### Enable and start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp.service
sudo systemctl status myapp
```

### Need a privileged port (<1024) without root? Give the binary only that one capability:
```bash
sudo setcap 'cap_net_bind_service=+ep' /opt/myapp/bin/myapp
getcap /opt/myapp/bin/myapp
```
## 4) Constrained Sudo for a Deploy User (optional)

Let deploy restart only specific services and read logs—nothing else.

### Create a sudoers drop-in:
```bash
echo 'deploy ALL=(root) NOPASSWD: /usr/bin/systemctl restart myapp.service, /usr/bin/systemctl status myapp.service, /usr/bin/journalctl -u myapp.service' \
| sudo tee /etc/sudoers.d/deploy-myapp >/dev/null
```
### Validate syntax BEFORE leaving the session:
```bash
sudo visudo -cf /etc/sudoers.d/deploy-myapp
```

#### Usage:
```bash
sudo -l -U deploy
sudo systemctl status myapp.service
sudo systemctl restart myapp.service
```
## 5) Files, Ownership, and ACLs

Keep ownership tight; grant write access only where the service needs it.

### Ownership to service user
```bash
sudo chown -R myapp:myapp /opt/myapp /var/lib/myapp /var/log/myapp
```
### Typical permissions
```bash
sudo chmod 750 /opt/myapp /var/lib/myapp /var/log/myapp
sudo chmod 640 /opt/myapp/config.yaml
```
### Optional fine-grained write with ACLs (when another group needs access)
```bash
sudo setfacl -m g:adm:rX /var/log/myapp
getfacl /var/log/myapp
```
## 6) User Services (advanced/optional)

If you prefer user-scoped systemd (no root units), enable lingering and run as the service user:

### Allow user services to start at boot without login
```bash
sudo loginctl enable-linger myapp
```
### As 'myapp' user:
```bash
sudo -u myapp mkdir -p ~/.config/systemd/user
sudo -u myapp cp myapp.user.service ~/.config/systemd/user/myapp.service
sudo -u myapp systemctl --user enable --now myapp.service
```
## 7) Verification Checklist
### Who runs the process?
```bash
ps -o user,group,comm,pid,ppid,tty,start,time -C myapp
```
### Systemd confirms the user:
```bash
systemctl show myapp.service -p User -p Group
```

### TCP listener not owned by root:
```bash
ss -tulpn | grep myapp
```
### Binaries aren’t world-writable:
```bash
namei -l /opt/myapp/bin/myapp
```

### Pass criteria
* Service runs as its dedicated user (myapp)
* No interactive shell for service users (/usr/sbin/nologin)
* Only required directories are writable
* setcap used instead of root for privileged ports (if needed)

## 8) Quick Reference (user & group commands)
### Create normal user (interactive)
```bash
sudo adduser <user>
```
### Create system user (no login)
```bash
sudo adduser --system --home /var/lib/<svc> --group --shell /usr/sbin/nologin <svc>
```
### Add to group(s)
```bash
sudo usermod -aG <group1>,<group2> <user>
```
### Change shell (e.g., lock login)
```bash
sudo usermod -s /usr/sbin/nologin <user>
```

### Lock or unlock account
```bash
sudo usermod -L <user>   # lock
sudo usermod -U <user>   # unlock
```
### Create group
```bash
sudo addgroup <group>
```
### Chown/chmod common patterns
```bash
sudo chown -R <user>:<group> /path
sudo chmod 750 /path
sudo chmod 640 /path/file
```
### Capabilities (privileged ports without root)
```bash
sudo setcap 'cap_net_bind_service=+ep' /path/to/bin
getcap /path/to/bin
```