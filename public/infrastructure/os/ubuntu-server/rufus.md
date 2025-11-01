---
title: Flashing a Drive with Rufus 
description: Steps to flash a new OS with ubuntu server with rufus and setting up ssh. 
published: true
date: 2025-11-01T16:05:42.174Z
tags: public, infrastructure, os, ubuntu-server, rufus
editor: markdown
dateCreated: 2025-11-01T15:07:26.541Z
---

# How To Flash With Rufus

## Purpose

This guide explains how to create a bootable USB drive using Rufus on Windows to install Ubuntu Server 24.04 LTS (or another supported OS) onto a physical device such as a mini-PC or homelab node.

## 1. Requirements
### Item	Description
* [Rufus	Download](https://rufus.ie/en/)
* [Ubuntu ISO	Obtain the 64-bit image](https://ubuntu.com/download/server)
* USB Drive	Minimum 8 GB (all data will be erased)
* Target Machine	Device you plan to install Ubuntu on (NUC, mini-PC, etc.)
* Network Info	Static IP, gateway, and DNS settings for later configuration
* Access Method	SSH client such as Windows Terminal or PuTTY
## 2. Steps to Flash

### 2.1 Insert USB Drive
* Plug the USB stick into your Windows computer.

### 2.2 Open Rufus
* No installation is required — double-click Rufus.exe.

### 2.3 Select the Drive
* Under Device, choose the correct USB drive.
* **Double-check capacity to avoid overwriting the wrong one**.

### 2.4 Choose ISO File

* Click Select, browse to your downloaded ubuntu-24.04-live-server-amd64.iso.
* Rufus automatically detects the image type.

### 2.5 Partition Scheme
* **For modern systems (UEFI)**: use GPT + UEFI (non-CSM).
* **For legacy BIOS**: choose MBR + BIOS (or UEFI-CSM).

### 2.6 Volume Label (optional)
* You may rename the volume, e.g., UBUNTU2404.
### Example Config
![rufussettings.png](/assets/images/rufus/rufussettings.png)
### 2.7 Click Start
* Confirm when Rufus warns that all data will be destroyed.
* The flashing process takes 5–10 minutes.

### 2.8 Verify
* When completed, Rufus shows “READY.”

### 2.9 Safely eject the drive

## 3. Boot and Install
### 3.1 Insert the Bootable USB into the target machine.
### Power on and enter BIOS/Boot Menu
* **Common keys: F2, F10, F12, or DEL**

### 3.2 Select USB Device to boot.
* Follow on-screen prompts for Ubuntu Server Setup:
* Choose Minimal Installation.
* Skip snaps, third-party drivers, and GUI.

### 3.3 Assign the desired hostname.
* **Example**: devmachine

### 3.4 (Optional) - Configure a static IP if desired or leave DHCP for now.

### 3.5 Wait for installation to complete and remove the USB when prompted.

## 4. First Boot & SSH Access
* Once installation finishes and the machine reboots, complete these steps to enable SSH so the node can be accessed remotely from your management computer.
* This configuration uses password authentication for the initial connection.
* **SSH key setup and hardening will be handled in later articles.**

### 4.1 Log In Locally
1. Log in at the console with the username and password you created during installation.
2. Update packages and install the SSH server:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y openssh-server
```
### 4.2 Enable and Start the SSH Service
* Ensure SSH starts now and on future boots:
```bash
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```
**You should see Active: active (running).**
If not, re-run the start command.

### 4.4 Confirm Password Authentication

By default, Ubuntu enables password logins.
the password is the password set for the account created on install
## 5. Set up SSH hosts For Remote Desktop in vscode
> For more information, see [SSH Hosts for Remote Desktop: vscode](/public/infrastructure/os/workstation/vscode/ssh)

### Terminal based (CMD/Powershell or Linux Terminal)
#### Example Template
```bash
ssh <yourusername>@<hostip>
#or
ssh <yourusername>@<yourhostname>
```
#### Filled out
* **username**: dev
* **ip**: 192.154.0.16
* **hostname**: devmachine
```bash
ssh dev@192.154.0.16
# or
ssh dev@devmachine.lcoal
```

If connection succeeds, you have completed the Rufus flash and initial installation steps.
You can now continue to the next section of the documentation: [