---
title: SSH Hosts With Vscode
description: How to Set up SSH Hosts With Vscode
published: true
date: 2025-11-01T15:57:44.528Z
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
### 2.2 Go to Extensions (left sidebar).
### 2.3 Search for “Remote - SSH”.
### 2.4 Install Remote - SSH (by Microsoft).
(Optional) Also install Remote - SSH: Editing Configuration Files — this makes editing ssh_config easier.

This adds a new icon on the left sidebar called Remote Explorer.

## 3. Create or Edit Your SSH Config
VS Code uses your normal OpenSSH config file, the same one you’d use in a terminal.
### 3.1 Verify if ssh folder exists
#### Windows Example
```bash
Windows: C:\Users\<you>\.ssh\config
```
#### Linux/mac example
```bash
Linux/macOS: ~/.ssh/config
```
### 3.2 If the file doesn’t exist, create it.


### 3.3 Add an entry like this:
#### Ubuntu Server with following credentials:
* **username**: dev
* **ip**: 192.154.0.16
* **hostname**: devmachine
#### Example
```bash
Host devmachine
    HostName 192.154.0.16
    User dev
    # For now we allow password auth. Later articles will swap to keys.
    # IdentityFile ~/.ssh/id_ed25519
```

### What this does:
* Host devmachine → the short name that shows up in VS Code
* HostName → the actual IP or DNS name
* User → the Linux user on the node

### You can add as many hosts as you want.

#### Example with multiple homelab nodes:
```bash
Host devnode1
    HostName 192.154.0.17
    User dev

Host k3s-worker1
    HostName 192.154.0.18
    User dev

Host utility-box
    HostName 192.154.0.19
    User dev
```

### 3.4 Save the file.

## 4. Connect from VS Code
### 4.1 Click Bottom left on the Remote Connection ICON
### 4.2 Click Connect to Host 
You should see the hosts you added (devnode1, k3s-worker1, etc.).

### 4.3 Click the one you want.
* VS Code will open a new window and try to SSH in.
* The first time, VS Code will install its remote server on the node — this is normal.

### 4.4 When prompted, enter your SSH password for that user.
If it connects, you’ll see “SSH: devmachine” in the lower-left corner of VS Code.

## 5. Open a Remote Folder
Once connected:

### 5.1 Click Open Folder…
### 5.2 Pick something like /home/dev
* VS Code will now act like it’s installed on the server — terminals, file browser, extensions.
* This is how you can let a friend/dev work on your devhost without giving them full desktop access.

## 6. (Optional) Trusted Host Keys
The very first time you connect, VS Code/SSH might say:

The authenticity of host ‘10.100.0.12’ can’t be established…

### 6.1 Choose Yes to add it to known_hosts.
This prevents MITM warnings later.

## 7. (Optional) Forwarding Ports
If the node is running something like:
```bash
go run main.go
# or
docker run -p 8080:8080 your-app
```

You can forward it to your local machine right from VS Code:

### 7.1 Open Command Palette → Remote-SSH: Forward a Port
### 7.2 Enter the remote port (e.g. 8080)
VS Code will give you a local URL to open in a browser
