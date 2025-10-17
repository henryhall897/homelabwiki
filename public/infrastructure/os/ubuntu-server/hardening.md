---
title: Ubuntu-Server Hardening Steps
description: Best Security practices to follow on a new ubuntu server install
published: true
date: 2025-10-17T22:43:57.203Z
tags: overview, infrastructure, public, os, ubuntu-server, hardening
editor: markdown
dateCreated: 2025-10-17T20:59:24.742Z
---

# Hardening Overview
## Purpose

The hardening process establishes the security baseline for every Ubuntu Server node in the homelab.
It ensures that each system follows a least-privilege, defense-in-depth model — minimizing the attack surface while maintaining stability and maintainability.

These steps are performed immediately after initial setup and before joining the node to the cluster.

> 🔗 See [Ubuntu Server setup](/public/infrastructure/os/ubuntu-server/setup)

## 1. Non-Root Operation

All applications and services run under dedicated, limited users instead of root.
This reduces the impact of misconfiguration or compromise and aligns with Kubernetes and container security best practices.

### Key Actions
* Create and assign non-root users or service accounts
* Restrict sudo usage to administrative accounts
* Verify all services use restricted User= directives (for systemd)

> [See Least Privilege & Non-Root Operation](/public/infrastructure/os/ubuntu-server/hardening/nonroot) 

## 3. Firewall (UFW)

The Uncomplicated Firewall (UFW) enforces a default deny policy on all nodes, permitting only the necessary management and application ports.

### Key Actions
* Set default deny inbound / allow outbound
* Allow SSH (22/tcp) and required service ports
* Apply host-specific rules for control-plane vs worker nodes
* Verify persistent enablement across reboots

> See Firewall Configuration (UFW)

### Wireguard
After Configuring a baseline Firewall, it is a good time to integrate wireguard and move ssh over to wireguard only. 
* Deny Regular SSH and only allow SSH over Wireguard VPN

> See [Wireguard Integration](/public/infrastructure/networking/wireguard)

## 4. Kernel & Sysctl Hardening

The kernel provides the last line of defense for process isolation and network behavior.
Applying secure defaults ensures predictable and restricted behavior across all nodes.

### Key Actions
* Apply hardened sysctl parameters for network and memory protection
* Drop unneeded kernel modules
* Enforce readOnlyRootFilesystem in workloads (via Kyverno)
* Define a restricted seccomp profile for user workloads

> See Kernel Hardening

## 5. Automated Updates

Automated updates ensure systems remain patched without manual intervention.
Security updates are installed silently, while package upgrades can be scheduled for maintenance windows.

### Key Actions
* Install and configure unattended-upgrades
* Enable automatic package list refreshes
* Validate logs to confirm updates are applied
* Reboot scheduling (if required for kernel patches)

> See Automatic Updates

## 6. Validation and Compliance

### Each hardened node should be tested to confirm that:
* Root login is disabled
* SSH key-only access is enforced
* UFW is active with the correct policy
* Unattended-upgrades service is running
* System logs show no failed service startups
* Future automation (e.g., via Ansible or Kyverno audits) can validate these states automatically.

## Summary

Hardening transforms a freshly installed Ubuntu Server into a secure, cluster-ready node.
By enforcing non-root operation, strong SSH policy, firewall boundaries, kernel restrictions, and automatic updates, the homelab maintains a consistent and trusted foundation across all environments