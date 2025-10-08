---
title: K3s Cluster Home Lab
description: High level Overview
published: true
date: 2025-10-03T00:24:32.077Z
tags: overview, homelab, index, architecture, summary
editor: markdown
dateCreated: 2025-10-03T00:24:21.701Z
---

# Homelab Overview
## Purpose
This homelab is designed to provide a learning environment for Kubernetes (k3s), networking, and self-hosting applications.  
It serves both as a personal playground for experimentation and a production-like environment for hosting services.

---

## Architecture
- **Cluster:** k3s running on Raspberry Pi and mini PC nodes.
- **Public Edge:** Hetzner Cloud VPS, acting as the exposed entry point for the internet.
- **Networking:** WireGuard tunnels secure traffic between the VPS and the homelab, abstracting the public IP and routing all inbound/outbound public traffic through the VPS.

---

## Benefits
- **Secure Exposure:** Home services remain private; only the VPS has a public IP.
- **Service Hosting:** Allows deployment of services such as Factorio and Minecraft servers accessible over the internet.
- **Learning Platform:** Practical environment to gain hands-on experience with:
  - Kubernetes (k3s)
  - Networking & VPNs
  - Service deployments & CI/CD
  - Security practices (isolation, ingress, policies)

---

## Current Services
- **Wiki.js** – Documentation platform for the homelab itself.
- **Factorio** – Multiplayer game server.
- **Minecraft (Aurora Bound Modpack)** – Modded server distribution and orchestration.
- **Golang To-Do App** – Custom microservice project deployed to the cluster.

---

## Future Plans
- Expand monitoring and observability (Prometheus, Grafana).
- Harden with security policies (Kyverno, Trivy scans).
- Add more automation with Ansible.
- Explore NAS integration for backups.