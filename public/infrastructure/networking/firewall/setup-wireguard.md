---
title: Firewall Setup 
description: Setting up firewall after confirming wireguard connection works. 
published: true
date: 2025-10-30T15:24:13.994Z
tags: public, infrastructure, networking, firewall, setup
editor: markdown
dateCreated: 2025-10-30T15:24:13.993Z
---



WireGuard Peers

Role	Description	WireGuard IP	Interface
VPS Gateway	Public entry point + router between peers	10.100.0.1	wg0
Spoke Node (e.g., K3s worker)	Private service host	10.100.0.2	wg0
User Interface	Developer/admin machine (your laptop or desktop)	10.100.0.3	wg0

Goal

Allow SSH and service access from the User Interface (10.100.0.3) to both:

The VPS (10.100.0.1) directly.

The Spoke (10.100.0.2) through the VPS.

Block all other public ingress.

Keep the WireGuard interface isolated from eth0 except where explicitly allowed.

🧱 1. VPS Firewall (Cloud Edge)

Purpose:
Public-facing system that terminates WireGuard and forwards traffic to the homelab’s private nodes.

Key rules:

# === Reset rules ===
sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing

# === Allow WireGuard port (from anywhere) ===
sudo ufw allow 51820/udp comment 'WireGuard VPN entry'

# === Allow SSH only from your WireGuard user interface (secure path) ===
sudo ufw allow from 10.100.0.3 to any port 22 proto tcp comment 'SSH from UI over VPN'

# === Allow forwarding between peers inside VPN ===
sudo ufw allow in on wg0 from 10.100.0.0/24 to any comment 'Allow internal WG traffic'
sudo ufw allow out on wg0 to 10.100.0.0/24 comment 'Allow internal WG traffic'

# === Drop all other inbound public traffic on eth0 ===
sudo ufw deny in on eth0 from any comment 'Drop all other public ingress'

sudo ufw enable


Optional (for packet forwarding):
In /etc/ufw/sysctl.conf, ensure:

net/ipv4/ip_forward=1


and in /etc/ufw/before.rules, add a forwarding section:

# allow forwarding for WireGuard
*filter
:ufw-before-forward - [0:0]
-A ufw-before-forward -i wg0 -j ACCEPT
-A ufw-before-forward -o wg0 -j ACCEPT
COMMIT

🧩 2. Spoke Node Firewall (Homelab Host)

Purpose:
Accept only connections from the VPS or authorized peers inside the VPN (not the public internet).

sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Accept inbound from VPS only
sudo ufw allow from 10.100.0.1 to any comment 'Allow traffic from VPS gateway'
# (Optional) allow admin access from UI peer routed through VPS
sudo ufw allow from 10.100.0.3 to any comment 'Allow indirect access from UI'

# Allow SSH internally only
sudo ufw allow in on wg0 to any port 22 proto tcp comment 'SSH via WireGuard'

# Block eth0 inbound completely
sudo ufw deny in on eth0 from any comment 'Block all public ingress'

sudo ufw enable

💻 3. User Interface (Developer Machine)

Purpose:
Client device that connects to the VPN and uses the VPS as its router to reach internal nodes.

sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow established replies
sudo ufw allow in on wg0 from 10.100.0.0/24 to any comment 'Allow VPN return traffic'

# Optional: restrict which nodes can be accessed
sudo ufw allow out on wg0 to 10.100.0.1 comment 'Allow VPS access'
sudo ufw allow out on wg0 to 10.100.0.2 comment 'Allow Spoke access via VPN'

sudo ufw enable

🕸️ Network Flow Diagram
%%{init: {'themeVariables': { 'fontSize': '14px', 'fontFamily': 'monospace' }}}%%
flowchart TD
    UI["User Interface (10.100.0.3)\nUFW client rules"]
    VPS["VPS Gateway (10.100.0.1)\nUFW + WireGuard + Forwarding"]
    SPOKE["Spoke Node (10.100.0.2)\nUFW internal rules"]

    UI <-->|"WireGuard tunnel"| VPS
    VPS <-->|"Forwarded peer traffic"| SPOKE

🧠 Behavior Summary
Connection	Path	Allowed?	Notes
UI → VPS	Direct via WireGuard	✅	SSH, Admin
UI → Spoke	Routed via VPS	✅	Through VPN only
Spoke → VPS	VPN heartbeat, cluster sync	✅	Internal control plane
Public → VPS	Denied except 51820/udp	🚫	VPN entry only
Public → Spoke	Always blocked	🚫	No direct ingress
🔒 Optional Hardening Additions

Fail2Ban on VPS for SSH and WireGuard rate limits.

UFW logging enabled selectively:

sudo ufw logging on
sudo ufw status verbose


Interface pinning (in on wg0) ensures no misrouted traffic via eth0.

Ansible automation can enforce these rules cluster-wide.