---
title: Operating Systems
description: Operating Systems used in homelab setup
published: true
date: 2025-10-17T01:43:04.031Z
tags: infrastructure, public, os
editor: markdown
dateCreated: 2025-10-17T01:43:04.031Z
---

# Operating Systems Overview
## Purpose

The homelab uses a mix of Linux and Windows systems, each chosen for their strengths in server reliability, developer tooling, and day-to-day usability. Together they form a flexible environment that supports both backend infrastructure and client-side interaction.

## 1. Server Platform — Ubuntu Server 24.04 LTS (64-bit)

Ubuntu Server is the foundation for nearly all cluster nodes.
It provides a stable, secure, and minimal Linux base optimized for long-term use and automation.

### Key Advantages
* Proven reliability — Built on the Debian ecosystem with long-term support (LTS) and predictable updates.
* Strong community — Vast documentation and support resources for troubleshooting and optimization.
* Tooling and ecosystem — Excellent compatibility with Docker, Kubernetes (K3s), Ansible, and CI/CD pipelines.
* Security-first configuration — Hardened to meet cluster security baselines.

### Core Configuration Principles
* Non-root operation and least-privilege roles
* Uncomplicated Firewall (UFW) with strict inbound policies
* Kernel-level hardening (sysctl, seccomp profiles)
* Automated unattended upgrades
* SSH hardened (key-based auth only, disabled root login)

🔗 See [Ubuntu Server Configuration — todo]

## 2. Personal and Interface Systems — Windows 11
The primary workstation and gaming PC runs Windows 11, serving as the interface node for development and daily use.

### Purpose
* Acts as the user interface to the homelab cluster
* Hosts VS Code Remote SSH sessions for development
* Runs Vscode and remote desktop to the devhost (development machine)
* Used for gaming, media, and personal tasks outside the homelab environment

While Windows is not used for hosting services, it plays a critical role as a management and interaction layer between the homelab and end-user environments.

🔗 See [Development Workstation Setup](./hardware/workstation)

## 3. Design Philosophy
### The overall OS strategy focuses on:
* **Minimal attack surface**: Reduce services, disable unneeded packages.
* **Consistency**: Uniform base images across all nodes for predictable deployments.
* **Maintainability**: Leverage LTS releases and automation to minimize manual upkeep.
* **Interoperability**: Seamless operation between Linux servers and Windows-based management tools.