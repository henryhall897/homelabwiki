---
title: Flashing a Drive with Rufus 
description: Steps to flash a new OS with ubuntu server with rufus and setting up ssh. 
published: true
date: 2025-11-01T15:07:26.541Z
tags: public, infrastructure, os, ubuntu-server, rufus
editor: markdown
dateCreated: 2025-11-01T15:07:26.541Z
---

How To Flash With Rufus

Path: /public/infrastructure/installation/how-to-flash-with-rufus
Breadcrumb: Infrastructure › Installation › Flashing Images › Rufus

Purpose

This guide explains how to create a bootable USB drive using Rufus on Windows to install Ubuntu Server 24.04 LTS (or another supported OS) onto a physical device such as a mini-PC or homelab node.

1. Requirements
Item	Description
Rufus	Download from https://rufus.ie
 — portable version recommended
Ubuntu ISO	Obtain the 64-bit image from Ubuntu Downloads

USB Drive	Minimum 8 GB (all data will be erased)
Target Machine	Device you plan to install Ubuntu on (NUC, mini-PC, etc.)
Network Info	Static IP, gateway, and DNS settings for later configuration
Access Method	SSH client such as Windows Terminal or PuTTY
2. Steps to Flash

Insert USB Drive
Plug the USB stick into your Windows computer.

Open Rufus
No installation is required — double-click Rufus.exe.

Select the Drive

Under Device, choose the correct USB drive.

Double-check capacity to avoid overwriting the wrong one.

Choose ISO File

Click Select, browse to your downloaded ubuntu-24.04-live-server-amd64.iso.

Rufus automatically detects the image type.

Partition Scheme

For modern systems (UEFI): use GPT + UEFI (non-CSM).

For legacy BIOS: choose MBR + BIOS (or UEFI-CSM).

Volume Label (optional)

You may rename the volume, e.g., UBUNTU2404.

Click Start

Confirm when Rufus warns that all data will be destroyed.

The flashing process takes 5–10 minutes.

Verify

When completed, Rufus shows “READY.”

Safely eject the drive.

3. Boot and Install

Insert the Bootable USB into the target machine.

Power on and enter BIOS/Boot Menu

Common keys: F2, F10, F12, or DEL.

Select USB Device to boot.

Follow on-screen prompts for Ubuntu Server Setup:

Choose Minimal Installation.

Skip snaps, third-party drivers, and GUI.

Assign the desired hostname (e.g., devnode1).

Configure a static IP if desired or leave DHCP for now.

Wait for installation to complete and remove the USB when prompted.

4. First Boot & SSH Access

Once installation finishes and the machine reboots:

# Example
ssh dev@10.100.0.12
# or
ssh dev@devnode1.local


If connection succeeds, you have completed the Rufus flash and initial installation steps.
You can now continue to the next section of the documentation: