---
title: Flash a USB drive with PI Imager for Headless Setup of Ubuntu Server
description: Instructions on how to setup a flash drive for a Ubuntu Server headless setup
published: true
date: 2025-11-01T16:12:38.972Z
tags: public, infrastructure, os, ubuntu-server, setup
editor: markdown
dateCreated: 2025-10-19T17:06:29.350Z
---

# Flashing Ubuntu Server with Raspberry Pi Imager — Headless Setup
## Purpose

This guide shows how to create a fully headless Ubuntu Server boot drive using Raspberry Pi Imager.
You’ll end up with an image that boots, joins the network, and is ready for SSH using your preloaded key—no monitor or keyboard required.

## 1. Requirements
* [Raspberry Pi Imager installed (v1.9.6 or newer)](https://www.raspberrypi.com/software/)
* [Micro SD card](https://www.amazon.com/uni-Reader-Adapter-Aluminum-Memory/dp/B087QG75L7/ref=sr_1_1_sspa?s=electronics&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY) or [USB Drive](https://www.amazon.com/dp/B09RG1TNM7) to store bootable image on

## 2. Flash the Image
1. Open **Raspberry Pi Imager**
2. **Choose Device**: Raspberry Pi 4 (or your model)
	* This option selects what image options will be displayed. For something like a mini PC, select newest Raspberry PI because That will show all the images. **This option does not effect boot config**.
2. **Choose OS**: Ubuntu Server 24.04 LTS (64-bit)
3. **Choose Storage**: your USB or SD card
### Example
![rppiimager.png](/assets/images/pimgr/rppiimager.png)

---

## 3. Configure Advanced Settings
1. Press **Next**
2. Press **EDIT SETTINGS**
### Example
![advancedimageroptions.png](/assets/images/pimgr/advancedimageroptions.png)

---

### General OS Settings:
3. **Set hostname**: Your Host Name Here. Tutorial **example**
4. **Set Username**: Your Username Here. Tutorial **example**
5. **Set Password**: Your Password Here. Tutorial **example**
	* [Bitwarden](https://bitwarden.com) is a solid password manager. Can Generate strong passwords and allow users to copy paste when needed. 
  
#### Example: 
![exampleconfig.png](/assets/images/pimgr/exampleconfig.png)

Homelab uses ethernet cabling. If Device is wireless, **configure wireless LAN**
### Services Settings:
6. **Enable SSH**:
7. Allow pub-key auth only. 

If key pair needs to be generated, then [generate keypair]()

8. Paste your **public** SSH key
9. Click Save → then Next → Write.
#### Example:
![oscustomize.png](/assets/images/pimgr/oscustomize.png)
## 4. Configure vscode on user interface for Remote Desktop
> [How to set up VScode for Remote Desktop](/public/infrastructure/os/workstation/vscode/ssh)
## Summary

Raspberry Pi Imager’s advanced settings let you automate first-boot provisioning without touching a keyboard.
It’s the simplest and most reproducible way to prepare new nodes for your homelab’s cluster environment.

## Finishing First time install
[Return to Setup](/public/infrastructure/os/ubuntu-server/setup)