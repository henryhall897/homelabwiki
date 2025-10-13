---
title: Networking Overview
description: Overview of Networking of homelab
published: true
date: 2025-10-13T15:24:28.007Z
tags: overview
editor: markdown
dateCreated: 2025-10-08T02:47:51.188Z
---

# Networking Overview

[WireGuard](./networking/wireguard) • [VPS Gateway](./networking/vps) • [Firewall](./networking/firewall) • [DNS](./dns/local-dns.md)

---

## Purpose

The networking layer serves as the **foundation of the homelab**, connecting private services, external access points, and multiple physical nodes under a unified and secure architecture.

It is designed to:
- **Isolate production workloads** from general home network traffic  
- **Protect the residential IP** from public exposure  
- **Enable secure, private communication** between nodes and external users  
- **Provide controlled ingress and egress** for public-facing applications  

---

## Core Components

### 1. WireGuard VPN
A lightweight, kernel-level VPN that forms the encrypted backbone of the homelab network.  
It connects home servers, VPS gateways, and remote devices under a single private subnet.  
> 🔗 See [WireGuard Overview](./networking/wireguard)

---

### 2. VPS Gateway
A cloud-hosted entry point that serves as the **public access layer** for all self-hosted services.  
The VPS connects to the homelab over WireGuard, routing public traffic through an isolated channel without revealing the home IP.  
> 🔗 See [VPS Gateway](./networking/vps)

---

### 3. Firewalls and Traffic Rules
Firewall policies enforce **north-south and east-west traffic control**.  
UFW, nftables, and Kubernetes network policies are used to limit connections by IP, interface, and namespace.  
> 🔗 See [Firewall Overview](./networking/firewall)

---

### 4. DNS and Domains
DNS plays a central role in routing both public and private traffic:
- **Public DNS:** Managed domains (e.g., `*.henryhalldev.com`) point to the VPS gateway.  
- **Private DNS:** Local resolution for cluster services and internal hostnames (e.g., `*.local`).  
> 🔗 See [DNS Configuration](./networking/dnsdomains)

---

## Security Considerations

- **IP exposure prevention:** Public ingress routed only through the VPS.  
- **Zero trust principle:** No direct LAN access to production workloads.  
- **Key management:** WireGuard keys rotated regularly (planned via Ansible).  
- **Defense in depth:** Firewall enforcement, namespace isolation, and restricted container privileges.  

---

## Summary

This networking model combines **WireGuard for encryption**, **VPS for isolation**, and **firewalls for segmentation** to create a secure, modular, and future-proof environment.

All public traffic is funneled through a hardened VPS, and all internal communication occurs through an encrypted private network, ensuring both security and flexibility across homelab services.

## Sub Articles  
* [wireguard](/public/infrastructure/networking/wireguard)
* [VPS](/public/infrastructure/networking/vps)
* [Firewall](/public/infrastructure/networking/firewall)
* [DNS and Domains](/public/infrastructure/networking/dnsdomains)