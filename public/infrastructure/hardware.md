---
title: Hardware Overview
description: Overview of the hardware used for this homelab
published: true
date: 2025-10-15T01:55:54.965Z
tags: overview, infrastructure, public, hardware
editor: markdown
dateCreated: 2025-10-15T01:55:54.965Z
---

# Hardware Overview
Purpose

The hardware layer forms the physical and virtual foundation of the homelab.
It defines the tangible computing resources that power Kubernetes workloads, host services, and enable secure network communication across environments.

This layer provides:

Scalability through modular nodes and replaceable components

Reliability via redundant power and network paths

Separation of Concerns between orchestration, workloads, and management systems

Upgrade Flexibility to adapt as requirements grow

Core Categories
1. Raspberry Pi Cluster

The backbone of the homelab’s Kubernetes environment.

Role	Model	RAM	Purpose	Notes
Control Plane	Raspberry Pi 4 B	8 GB	Hosts k3s server and core control plane services	Boots from external SSD for reliability
Worker Node #1	Raspberry Pi 4 B	8 GB	Handles application pods and offloaded workloads	Same configuration as control plane
Ansible Node	Raspberry Pi 4 B	4 GB	Dedicated for automation, provisioning, and management tasks	Planned to use SSD as OS drive

The Raspberry Pi cluster provides a power-efficient and fully self-contained Kubernetes environment for testing and production services.

🔗 See Cluster Architecture

2. Mini-PC Nodes

Small-form-factor systems providing additional compute power for heavier workloads.

Primary Mini PC — Hosts game servers (Minecraft and Factorio), GitHub repos, and management scripts.

Planned Upgrade — A new mini PC to assume control-plane duties for production services, offloading the Raspberry Pis to be dedicated workers.

This separation improves reliability, resource allocation, and makes upgrading or rebooting less disruptive.

🔗 See Mini PC Setup

3. Virtual Infrastructure (VPS)

Although not physically owned, the Hetzner VPS serves as a crucial hardware-equivalent component — the public edge of the homelab.

Purpose: Acts as the hub in the WireGuard overlay network

Functions: Public ingress, reverse proxy routing, and secured tunnel endpoint

Location: Remote Datacenter (Hetzner Cloud)

Benefits: Provides static public IP without exposing home network

Treat this as virtual hardware: it behaves like a remote physical node but is rented compute capacity rather than owned silicon.

🔗 See VPS Gateway

4. Personal and Development Systems

Devhost Workstation: Used for local development, VS Code Remote sessions, and cluster orchestration.

Laptop: Used for remote work and access to services (e.g., Wiki.js UI and SSH sessions).
These systems serve as interface nodes between the homelab infrastructure and end-user environments.

🔗 See Development Workstation Setup

5. Networking and Power Infrastructure

Supporting devices that tie everything together:

Router: Asus Router (central LAN gateway)

Switches: Managed Gigabit Switches for cluster and mini PC interconnectivity

Cabling: Cat 6 Ethernet runs throughout for stability and speed

Power Management: Surge-protected power strips and organized cable layout

⚙ MoCA Adapters (Deprecated Experiment)

Previously tested for bridging Ethernet over coax, but deprioritized due to:

Interference and signal degradation over long coax runs

Inconsistent throughput compared to direct Ethernet

Complexity in troubleshooting and routing loops

Future Upgrades
Planned Upgrade	Purpose
New Mini PC Server	Serve as primary control plane for heavier workloads and container hosting
Dedicated NAS	Centralized storage for backups and persistent volumes
SSD Reassignment	Either reused for Ansible OS or remain with Pi control plane
Network Cleanup	Replace MoCA links with dedicated Ethernet where possible
Summary

The homelab hardware layer integrates local compute, virtual infrastructure, and network support devices into a cohesive, modular system.
It provides a foundation that is scalable, secure, and adaptable, ready to expand as new nodes, NAS storage, or dedicated services come online.

Each physical or virtual node serves a distinct role but contributes to a unified ecosystem focused on performance, isolation, and reproducibility.