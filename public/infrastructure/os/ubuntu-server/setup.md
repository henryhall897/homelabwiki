---
title: Ubuntu Server 24 LTS Setup
description: Baseline Fresh Ubuntu Setup for homelab node
published: true
date: 2025-10-17T16:31:47.024Z
tags: setup, infrastructure, public, os, ubuntu-server
editor: markdown
dateCreated: 2025-10-17T02:10:32.312Z
---

# Cluster Node Setup
## Purpose

This guide outlines the standard procedure for provisioning a new Ubuntu Server node within the homelab.
All nodes—whether control plane, worker, or utility—follow the same minimal baseline to ensure consistency, reproducibility, and security.

## 1. Prerequisites

### Before installation:
* Download [Ubuntu Server 24.04 LTS (64-bit) image.](https://ubuntu.com/download/server)
* [Micro SD card](https://www.amazon.com/uni-Reader-Adapter-Aluminum-Memory/dp/B087QG75L7/ref=sr_1_1_sspa?s=electronics&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY) or [USB Drive](https://www.amazon.com/dp/B09RG1TNM7) to store bootable image on
* [Raspberry PI Imager](https://www.raspberrypi.com/software) for flashing to the USB Drive or SD card

> [How To Flash a USB/MicroSD with Pi Imager](/public/infrastructure/os/ubuntu-server/flashusb)

### Have access to:
* A laptop or PC or any computer that can SSH to new machine
* Network information (static IP, gateway, DNS)

## 2. Installation

### Select Minimal Installation
During setup, choose Minimal install to avoid unnecessary packages and GUI components.
Do not install third-party drivers or snaps at this stage.

This naming convention simplifies Ansible inventory and WireGuard peer management.

## 3. Post-Installation Configuration
After the first boot, log in locally or via SSH
### SSH
```bash
ssh example@example.local
```
**OR**
```bash
ssh example@<node-ip>
```
Then perform the following:
### 3.1 System Update
```bash
sudo apt update && sudo apt full-upgrade -y
```
### 3.2 Enable Unattended Security Updates
```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

### 3.3 Configure SSH
```bash
sudo nano /etc/ssh/sshd_config
```
#### Ensure the following options are set:

```bash
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

#### Restart SSH:
```bash
sudo systemctl restart ssh
```
### 3.4 Configure Firewall (UFW)

```bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw status verbose
```

## 4. Core Tools and Utilities
### Install essential maintenance and monitoring packages:
```bash
sudo apt install -y curl git fail2ban htop net-tools jq
```

### Optional (depending on role):
```bash
sudo apt install docker.io -y
sudo systemctl enable docker
```
## 5. Validation
Run quick system checks to confirm baseline compliance:

### Verify UFW
```bash
sudo ufw status
```
### Confirm root login disabled
```bash
sudo grep "PermitRootLogin" /etc/ssh/sshd_config
```
### Test unattended-upgrades
```bash
sudo systemctl status unattended-upgrades
```

## 7. Snapshot and Add to Cluster
### Once validated:
1. Create a snapshot or SD backup of the clean node image for reuse.
2. Join to the cluster using your K3s installation or Ansible automation.

> 🔗 See [K3s Node Join Procedure — todo]
🔗 See [Ansible Automation Overview — todo]

## Summary
* The Cluster Node Setup ensures each Ubuntu Server instance is:
* Minimal — no unneeded packages
* Secure — non-root, UFW, SSH key-only
* Consistent — uniform configuration across all nodes

This standardization guarantees reliability, simplicity, and security throughout the homelab infrastructure.

> This is a good baseline. Next Step would be [Wireguard Integration](/public/infrastructure/networking/wireguard)