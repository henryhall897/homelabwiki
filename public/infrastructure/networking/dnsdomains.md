---
title: DNS and Domains
description: DNS and Domains are crucial for getting public web services easily available to end users
published: true
date: 2025-10-11T00:27:32.074Z
tags: overview, networking, infrastructure, public, dns
editor: markdown
dateCreated: 2025-10-11T00:27:32.074Z
---

# DNS and Domains

## Purpose

The **DNS and domain layer** provides the public-facing identity of the homelab and handles name resolution for both external and internal services.

By using a single domain name and strategically defined subdomains, all homelab services can be accessed in a **unified**, **human-readable way** — whether they are hosted locally, in the cloud, or inside the **Kubernetes cluster**.

## Domain Providers

**Domains** can be registered and managed through various providers, each offering different pricing, reliability, and integration options.
### Common examples include:
#### Cloudflare:	
**Features**:
* Free DNS 
* Global CDN
* DDoS protection 
* SSL/TLS management
* Workers
* R2 storage

**Summary**:
Excellent for technical users. Integrates well with Kubernetes ingress, Traefik, and self-hosted services.
#### Google Domains (now under Squarespace)	
**Features**: 
* Simple management
* easy Gmail/Workspace integration	

**Summary**:
* Less flexible for advanced configurations.
#### Namecheap
**Features**:
* Affordable 
* straightforward UI
* good support	

**Summary**:
* Great for basic DNS setups.
#### Porkbun	
**Features** 
* Budget-friendly
* free WHOIS privacy	Reliable for simple domain ownership.

**Summary**:
Solid choice especially on a budget
#### AWS Route 53	
**Features**:
* Scalable
* API-driven 
* DNS with health checks
* private hosted zones	

**Summary**:
* Enterprise-grade but more complex.
## Why I Chose Cloudflare

### I use Cloudflare as my domain registrar and DNS provider because:
* It’s a trusted, security-focused company with one of the largest global DNS networks.
* It provides free DNS hosting with low latency and built-in protection against common network attacks.
* It integrates tightly with modern web stacks — including support for CNAME flattening, custom records, and easy API automation.
* It offers additional services I may use in the future, such as Cloudflare R2 (object storage), Zero Trust tunnels, and page rules for traffic management.
* Cloudflare also has an outstanding record of defending against DDoS attacks and large-scale outages, making it a robust choice for homelab DNS.

## Subdomains and Unified Access
A single registered domain can host multiple services using subdomains, allowing consistent and organized access:

### Examples in homelab
#### Factorio	
**factorio.henryhalldev.com** - Public Factorio server proxied through VPS
#### Minecraft	
**mc.henryhalldev.com** - Dockerized Minecraft instance
#### Wiki.js	
[wiki.henryhalldev.com](https://wiki.henryhalldev.com) - 	Knowledge base and documentation

This structure scales naturally as new applications are added to the homelab, and it keeps service URLs readable and consistent.

## HTTPS and Proxying

Cloudflare supports SSL/TLS termination and proxying (orange cloud mode), but this depends on where the service terminates HTTPS:

### Full (Strict) mode: 
Can secure traffic end-to-end when your VPS or Traefik ingress provides a valid certificate (e.g., via Let’s Encrypt).

### Proxying (orange cloud): 
Can hide your VPS’s public IP, route requests through Cloudflare’s network, and offer caching and DDoS protection.

### DNS-only (gray cloud): 
Simply resolves DNS and passes traffic directly to the VPS — this is often required for custom TCP/UDP services (like Factorio or Minecraft), since Cloudflare can’t proxy those ports.

Unfortunately, it is not possible to proxy Factorio’s raw game traffic through Cloudflare, but other web services can proxy HTTPS services through Cloudflare with SSL and caching enabled.

## Private DNS
For internal name resolution, private DNS handles traffic within the homelab network:

### Internal hostnames: 
such as **devhost.local** or **k3s-master.local**

### Cluster service domains 
like **postgres.default.svc.cluster.local**

### Optional integration with CoreDNS or systemd-resolved for local routing rules

Private DNS ensures that local nodes and containers communicate using private IPs over WireGuard, without exposing internal networks to the public internet.

## Summary

Cloudflare provides **DNS management**, **HTTPS proxying**, and a suite of security tools ideal for homelab use.

**One domain** is enough for **all public services**, thanks to subdomains that map to different apps or nodes.

HTTPS proxying is supported for web services but not for custom TCP/UDP game ports.

Private DNS resolves internal services within the WireGuard network and Kubernetes cluster.

🔗 Next: DNS Configuration

🔗 Related: [VPS Gateway](./vps)
 •