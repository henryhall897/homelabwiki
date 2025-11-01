---
title: SSH Hosts With Vscode
description: How to Set up SSH Hosts With Vscode
published: true
date: 2025-11-01T15:42:18.510Z
tags: public, infrastructure, os, workstation, vscode, ssh
editor: markdown
dateCreated: 2025-11-01T15:42:18.510Z
---

# Set Up SSH Hosts for Remote Development in VS Code

## Purpose

This article shows how to set up Visual Studio Code to connect to your homelab nodes (Ubuntu Server, Pi, mini-PC, etc.) over SSH using the Remote - SSH extension.
This assumes the node already allows SSH with password auth (from your previous step). Later articles can upgrade this to key-based + hardening.

## 1. Prerequisites
### You should already have:
* A running node (Ubuntu Server 24.04) with SSH enabled
* The IP or hostname of the node
### Example
#### Ubuntu Server with following credentials:
* **username**: dev
* **ip**: 192.154.0.16
* **hostname**: devmachine

### A desktop/laptop with:
* VS Code installed
* Internet access to install extensions
* Optional (nice to have):
* .ssh folder on your workstation:
#### Windows Example
```bash
Windows: C:\Users\<you>\.ssh\
```
#### Linux/mac example
```bash
Linux/macOS: ~/.ssh/
```
## 2. Install the VS Code Extension

### 2.1 Open VS Code.

Go to Extensions (left sidebar).

Search for “Remote - SSH”.

Install Remote - SSH (by Microsoft).

(Optional) Also install Remote - SSH: Editing Configuration Files — this makes editing ssh_config easier.

This adds a new icon on the left sidebar called Remote Explorer.

3. Create or Edit Your SSH Config

VS Code uses your normal OpenSSH config file, the same one you’d use in a terminal.

Windows: C:\Users\<you>\.ssh\config

Linux/macOS: ~/.ssh/config

If the file doesn’t exist, create it.

Add an entry like this:

Host devnode1
    HostName 10.100.0.12
    User dev
    Port 22
    # For now we allow password auth. Later articles will swap to keys.
    # IdentityFile ~/.ssh/id_ed25519


What this does:

Host devnode1 → the short name that shows up in VS Code

HostName → the actual IP or DNS name

User → the Linux user on the node

Port → usually 22 (unless you changed SSH)

You can add as many hosts as you want.

Example with multiple homelab nodes:

Host devnode1
    HostName 10.100.0.12
    User dev

Host k3s-worker1
    HostName 10.100.0.21
    User dev

Host utility-box
    HostName 10.100.0.40
    User dev


Save the file.

4. Connect from VS Code

In VS Code, press F1 (or Ctrl+Shift+P).

Type: Remote-SSH: Connect to Host…

You should see the hosts you added (devnode1, k3s-worker1, etc.).

Click the one you want.

VS Code will open a new window and try to SSH in.

The first time, VS Code will install its remote server on the node — this is normal.

When prompted, enter your SSH password for that user.

If it connects, you’ll see “SSH: devnode1” in the lower-left corner of VS Code.

5. Open a Remote Folder

Once connected:

Click Open Folder…

Pick something like /home/dev/github-repos or /home/dev

VS Code will now act like it’s installed on the server — terminals, file browser, extensions.

This is how you can let a friend/dev work on your devhost without giving them full desktop access.

6. (Optional) Trusted Host Keys

The very first time you connect, VS Code/SSH might say:

The authenticity of host ‘10.100.0.12’ can’t be established…

Choose Yes to add it to known_hosts.
This prevents MITM warnings later.

7. (Optional) Forwarding Ports

If the node is running something like:

go run main.go
# or
docker run -p 8080:8080 your-app


You can forward it to your local machine right from VS Code:

Open Command Palette → Remote-SSH: Forward a Port

Enter the remote port (e.g. 8080)

VS Code will give you a local URL to open in a browser

This is great for testing services inside the cluster network from your laptop.

8. Notes for Your Homelab Style

You can store all homelab nodes in one ~/.ssh/config and just pick them in VS Code.

You can use WireGuard hostnames (like 10.100.0.x) the exact same way — as long as your laptop is on WG.

Later, when you add Ansible and key-based auth, this article stays valid — only the ssh_config entries change