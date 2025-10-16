---
title: Workstation Setup
description: How the homelab is managed
published: true
date: 2025-10-16T03:54:56.283Z
tags: infrastructure, public, hardware, workstation
editor: markdown
dateCreated: 2025-10-16T03:54:56.283Z
---

# Personal and Development Systems

These systems act as operator interfaces — the bridge between user workflows and the homelab infrastructure.
They are not part of the Kubernetes cluster itself but provide control, access, and orchestration capabilities.
## 1. Personal Computer - User Interface - (Windows)
> Author Note: For general PC use, I prefer windows. As a gamer, windows is simply the best platform and I am used to using it. However, For devleopment, Linux is hands down the best. 
### This solution gives best of both worlds:
* Windows UI that I am comfortable with
* Full Native Linux Development through VScode Remote Desktop
## 2. Devhost Workstation
 The Devhost is the primary development and management node for the homelab.

### Purpose
* Local development environment for Go, Docker, and Helm projects
* Manages Kubernetes, Hosts Test Enviroments, Github Repos, and Servers. 
* Platform for building, testing, and staging code before deployment
* Host for lightweight scripts and CLI tooling (e.g., kubectl, helm, k9s)
### Future Upgrade
New PC that can decouple devhost from some of its use cases
* Move server hosting out so its purely a manager PC

### Notes
* Configured with WireGuard peer access to all internal nodes
* Will later delegate automation tasks to the Ansible node, reducing manual SSH use
* All Nodes besides User Interface rune Ubuntu Server

## 3. Laptop
The laptop functions as the mobile access point for both internal and remote management.

### Purpose
* Connect to homelab services via WireGuard (Wiki.js, Factorio, etc.)
* Perform administrative work while away from the home network
* Serve as a secure thin-client for SSH and browser-based management

### Notes
* Uses VS Code Remote for transient configuration changes
* No persistent dependencies; can be replaced or rotated easily
### Role in Infrastructure
## Together, these systems provide:
* Operator interface layer between users and servers
* Development sandbox for local iteration before deployment
* Fallback access for troubleshooting when automation or cluster services are unavailable