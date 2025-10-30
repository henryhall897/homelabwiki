---
title: Wireguard Overview
description: Wireguard Overview
published: true
date: 2025-10-30T15:33:43.371Z
tags: overview, networking, wireguard
editor: markdown
dateCreated: 2025-10-19T17:05:09.131Z
---

# WireGuard Overview

[Overview](/public/infrastructure/networking/wireguard) - [Setup](/public/infrastructure/networking/wireguard/setup) - [Troubleshooting](/public/infrastructure/networking/wireguard/troubleshooting) - [Commands](/public/infrastructure/networking/wireguard/commands) - [Ansible Automation]() #Todo add ansible article+link

---

## Objectives
The primary goals of implementing WireGuard within the homelab environment are:

- **Secure network isolation** between production deployments and the general home LAN  
- **Protect home users and devices** that share the same public internet connection  
- **Establish a controlled ingress path** for external users and services accessing self-hosted applications  

---

## Role in the Network Architecture

WireGuard forms the **private backbone** of the homelab network.  
It is responsible for:
- Creating an encrypted tunnel between the **home cluster** and a **cloud VPS gateway**
- Providing internal connectivity between nodes and services across sites
- Allowing controlled ingress through the VPS without exposing the home IP

While WireGuard handles the *private side* of the equation, **public traffic isolation** is achieved through the VPS gateway.  

> ⚠️ **Note:**  
> The VPN alone does **not** prevent your home IP from being exposed.  
> To solve that problem, a cloud VPS acts as the public access point for all inbound traffic.  
> See [VPS Gateway](./vps) for a detailed explanation of this design.

---
## Topology Example
```mermaid
%%{init: {'flowchart': {'padding': 24}}}%%
flowchart TD
  subgraph WG["WireGuard Hub-and-Spoke (wg0)"]
    P1["Peer 1 — VPS (Hub)\nWG IP: 10.100.0.1\nDirect peers with both (2 tunnels) and allows both IPs\n\n"]
    P2["Peer 2 — Server Node (Spoke)\nWG IP: 10.100.0.2\nService: SSH\n\n[Peer (VPS) AllowedIPs]\n- 10.100.0.1/32 (Hub)\n- 10.100.0.3/32 (User Interface Spoke)\n\n"]
    P3["Peer 3 — User Interface (Spoke)\nWG IP: 10.100.0.3\nClient: ssh\n\n[Peer (VPS) AllowedIPs]\n- 10.100.0.1/32 (hub itself)\n- 10.100.0.2/32 (Peer 2 via hub)\n\n"]
  end
  P3 ==>|"ssh to 10.100.0.2\n(route to hub per AllowedIPs)\n\n"| P1
  P1 ==>|"forward to 10.100.0.2\n\n"| P2
  P2 ==>|"reply via hub\n\n"| P1
  P1 ==>|"forward reply\n\n"| P3
```
## Why WireGuard?

WireGuard is a **modern VPN protocol** built directly into the Linux kernel, designed for **speed, simplicity, and strong cryptography**.  
Its compact codebase (a few thousand lines) makes it easier to audit and maintain compared to legacy VPNs such as OpenVPN or IPSec.

### Key Advantages
- **Kernel-level performance:** Low latency and high throughput, even on small devices like Raspberry Pi or VPS nodes  
- **Modern cryptography:** Uses state-of-the-art algorithms and key exchange mechanisms  
- **Minimal configuration:** Simple, predictable syntax  
- **Flexible topology:** Supports hub-and-spoke, mesh, or hybrid peer layouts  
- **Granular control:** Each peer has a unique key pair and can include an additional preshared key for extra security  

### Trade-offs
WireGuard’s design favors simplicity over automation:
- No built-in key or identity management system  
- Each peer must have its own public/private key pair  
- Configuration distribution and rotation must be handled manually  
- Topology and routing require explicit planning  

> 💡 **Personal Insight**  
> Manually setting up WireGuard provided valuable understanding of VPN topology, routing, and key rotation.  
> While Tailscale (a service built on WireGuard) automates most of this, the manual process offers deeper insight into network security and design.

---

## How It Fits Together

In the overall architecture:

1. **WireGuard** creates a secure private network between the home servers and the VPS gateway.  
2. **The VPS** serves as the *only* public entry point, forwarding traffic through the VPN to internal services.  
3. **Kubernetes namespaces and firewalls** maintain isolation between production, staging, and local-only resources.  

This combination keeps the **home IP hidden**, the **network segmented**, and **access tightly controlled**.

> 🔗 For details on the VPS’s public routing and isolation layer, see [VPS Gateway](./vps).

---

## Future Automation with Ansible

Manual configuration can become cumbersome as the network grows.  
Integrating **Ansible** will address many of these limitations by:

- Automating key generation and distribution  
- Applying consistent WireGuard configuration templates to each node  
- Enabling one-command onboarding for new devices  

> 🔗 See [Ansible Automation](./ansible-automation.md) for details on this future implementation.

---

## Comparison: WireGuard vs. Tailscale

Before adopting WireGuard, **Tailscale** was used for private server connections.  
Tailscale manages keys, routing, and peer discovery automatically — simplifying setup but reducing fine-grained control.

| Aspect | WireGuard | Tailscale |
|--------|------------|-----------|
| Setup Complexity | Manual setup & routing | One-click device registration |
| Key Management | Manual | Automated via Tailscale control plane |
| Topology | Customizable (hub, mesh, hybrid) | Automatic full mesh |
| Control | Full transparency and flexibility | Simplified, less customizable |
| Best For | Power users, automation, and learning | Quick deployments, smaller homelabs |

### Summary
- **WireGuard** – Best for users seeking full control and understanding of VPN internals.  
- **Tailscale** – Ideal for simple, managed setups where convenience is the priority.  

> 💬 **Personal Note**  
> Tailscale’s convenience makes it a solid option for many homelab environments.  
> However, with Ansible automation planned, the advantages of Tailscale become less significant for this setup.

---

## Verdict

WireGuard serves as the **secure transport layer** for the homelab’s private network.  
Combined with a **cloud-hosted VPS gateway** and planned **Ansible automation**, it creates a reliable, high-performance, and privacy-preserving foundation that separates public traffic from private infrastructure.

---

## Wireguard Articles
[Overview](/public/infrastructure/networking/wireguard) - [Setup](/public/infrastructure/networking/wireguard/setup) - [Troubleshooting](/public/infrastructure/networking/wireguard/troubleshooting) - [Commands](/public/infrastructure/networking/wireguard/commands)

---