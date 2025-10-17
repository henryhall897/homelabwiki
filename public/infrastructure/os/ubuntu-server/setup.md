---
title: Ubuntu Server 24 LTS Setup
description: Baseline Fresh Ubuntu Setup for homelab node
published: true
date: 2025-10-17T14:57:30.843Z
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

> Pre Install Setup - todo

### Have access to:
* A laptop or PC or any computer that can SSH to new machine
	* Raspberry PI Imager lets a user be created, SSH Enabled, and Keys added 
* SSH key pair (public key ready to import)
* Network information (static IP, gateway, DNS)
* WireGuard peer configuration (if applicable)
	* See [Wireguard Setup](/public/infrastructure/networking/wireguard/setup)

## 2. Installation

### Select Minimal Installation
During setup, choose Minimal install to avoid unnecessary packages and GUI components.
Do not install third-party drivers or snaps at this stage.

### Create Non-Root User
Define a username such as dev or clusteradmin.

Assign a strong password (temporary; will disable password auth later).

Ensure “Allow this user to administer the system” is checked to grant sudo.

Enable OpenSSH Server
When prompted, select Install OpenSSH Server to allow remote management after setup.

Set Hostname
Use a consistent naming scheme:

cluster1     → Control Plane
devnode2     → Worker Node
ansible      → Automation Node


This naming convention simplifies Ansible inventory and WireGuard peer management.

3. Post-Installation Configuration

After the first boot, log in locally or via SSH and perform the following setup:

3.1 System Update
sudo apt update && sudo apt full-upgrade -y

3.2 Enable Unattended Security Updates
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades

3.3 Configure SSH
sudo nano /etc/ssh/sshd_config


Ensure the following options are set:

PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes


Restart SSH:

sudo systemctl restart ssh

3.4 Configure Firewall (UFW)
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw status verbose

3.5 Add SSH Public Key

From your workstation:

ssh-copy-id dev@<node-ip>


Or manually append your key to:

~/.ssh/authorized_keys

4. Network Configuration
4.1 Static IP (Optional if Using DHCP Reservation)

Edit the Netplan configuration file:

sudo nano /etc/netplan/50-cloud-init.yaml


Example:

network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.20/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]


Apply changes:

sudo netplan apply

4.2 WireGuard Tunnel

If the node connects via VPN:

sudo apt install wireguard -y
sudo wg genkey | tee private.key | wg pubkey > public.key
sudo mv private.key /etc/wireguard/keys/
sudo mv public.key /etc/wireguard/keys/
sudo chmod 600 /etc/wireguard/keys/private.key


Create /etc/wireguard/wg0.conf based on your cluster’s template.
🔗 See [WireGuard Setup — todo]

5. Core Tools and Utilities

Install essential maintenance and monitoring packages:

sudo apt install -y curl git fail2ban htop net-tools jq


Optional (depending on role):

sudo apt install docker.io -y
sudo systemctl enable docker

6. Validation

Run quick system checks to confirm baseline compliance:

# Verify UFW
sudo ufw status

# Confirm root login disabled
sudo grep "PermitRootLogin" /etc/ssh/sshd_config

# Test unattended-upgrades
sudo systemctl status unattended-upgrades

# Confirm WireGuard status
sudo systemctl status wg-quick@wg0

7. Snapshot and Add to Cluster

Once validated:

Create a snapshot or SD backup of the clean node image for reuse.

Join to the cluster using your K3s installation or Ansible automation.

🔗 See [K3s Node Join Procedure — todo]
🔗 See [Ansible Automation Overview — todo]

Summary

The Cluster Node Setup ensures each Ubuntu Server instance is:

Minimal — no unneeded packages

Secure — non-root, UFW, SSH key-only

Consistent — uniform configuration across all nodes

Ready for Automation — easily managed through Ansible and WireGuard

This standardization guarantees reliability, simplicity, and security throughout the homelab infrastructure.