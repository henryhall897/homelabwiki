---
title: Infrastructure Overview
description: Infrastructure of Homelab 
published: true
date: 2025-10-15T01:25:46.413Z
tags: overview, infrastructure, public
editor: markdown
dateCreated: 2025-10-13T15:38:31.112Z
---

# Infrastructure Overview

## Purpose

* The infrastructure layer provides the physical and virtual foundation of the homelab.
* It brings together hardware, operating systems, orchestration tools, storage, and monitoring systems under a single cohesive environment.

## This layer is designed to:
* Deliver a stable and reproducible platform for all services
* Support horizontal scaling through modular nodes and containers
* Maintain strict isolation between production, test, and management workloads
* Enable automation and observability across all components

## Core Components
### 1. Hardware
The **physical backbone** of the homelab, spanning local and remote nodes:

* Raspberry Pi cluster (control-plane and worker nodes)
* Mini-PC nodes for heavier workloads
* Hetzner VPS serving as the public access gateway
* Devhost workstation for development and orchestration

Network devices such as routers, MoCA adapters, and managed switches

🔗 See Hardware Overview - todo add link

### 2. Operating Systems

Each node runs a *hardened, minimal Linux base* — primarily **Ubuntu Server** — configured for reliability and low resource overhead.

#### Key configurations include:
* Non-root operation and least-privilege user roles
* UFW and kernel-level network security
* Automated updates and SSH hardening

🔗 See Operating Systems Overview - todo add article + link

### 3. [Networking](/public/infrastructure/networking)

The **private communication backbone** that links all nodes and services.
It uses WireGuard VPN to form an encrypted overlay network connecting the VPS, cluster nodes, and remote devices.

#### Core elements include:
* [WireGuard VPN](./infrastructure/networking/wireguard)
* [VPS Gateway](./infrastructure/networking/vps)
* [Firewall and traffic segmentation](./infrastructure/networking/firewall)
* [DNS and domain management](./infrastructure/networking/dnsdomains)

> 🔗 See [Networking Overview](/public/infrastructure/networking)

## 4. Orchestration
* The control layer responsible for deploying and managing workloads.
* K3s — lightweight Kubernetes distribution optimized for edge nodes
* Helm — templated deployments for repeatable, versioned applications
* Traefik — ingress routing, HTTPS termination, and load balancing
* Kyverno — policy enforcement and container hardening

🔗 See Orchestration Overview - todo

## 5. Storage
* Persistent data management and backup systems ensuring reliability across restarts and node updates.
* Components include:
* Persistent volumes for databases and applications
* External storage integration (e.g., NAS, Cloudflare R2)
* Automated backup and sync mechanisms (rsync, RClone)

🔗 See Storage Overview - todo

## 6. Automation
* Tools and scripts that enable reproducible setup and configuration across the entire homelab.
* Ansible for provisioning servers, WireGuard peers, and firewall rules
* Custom shell orchestrators for app-specific automation (e.g., Factorio, Minecraft)
* Plans for integration with GitHub Actions for CI/CD and updates

🔗 See Automation Overview

## 7. Monitoring
* The observability layer that tracks system health, performance, and uptime.
* Node metrics (CPU, memory, disk usage)
* Centralized logging and alerting
* Future integration with Prometheus, Grafana, or Loki

🔗 See Monitoring Overview

## Security Considerations
* The infrastructure layer is built around a zero-trust model and enforces:
* Non-root operation across all workloads
* Read-only root filesystems and seccomp enforcement where possible
* Network isolation via namespaces, subnets, and firewalls
* Regular key rotation and access control reviews
* Each component is designed to operate independently yet securely, forming a defense-in-depth architecture.

## Summary

The homelab’s infrastructure integrates hardware, networking, orchestration, and automation into a modular, secure, and maintainable platform.
Every layer — from WireGuard networking to Kubernetes orchestration — is built for transparency, reproducibility, and long-term scalability.

As new components are added, they can seamlessly extend this foundation without compromising reliability or security.

---