---
title: Wireguard Setup
description: How to do initial wireguard setup
published: true
date: 2025-10-10T15:31:42.263Z
tags: wireguard, networking, setup
editor: markdown
dateCreated: 2025-10-10T15:28:50.620Z
---

# WireGuard Setup (Ubuntu)

**Scope**: Ubuntu 22.04+ on a VPS (listener/server) and one or more peers (laptops, home nodes, etc.).
**Goal**: install WireGuard, generate keys (including an optional preshared key), pick your listener, and bring up your first tunnel with tight endpoint + routing controls.

## 1. Decide your topology
* **Listener (Server)**: your VPS should listen on a stable public IP/DNS and port 51820/udp (or any unused UDP port).
* **Peers**: everything else (home dev box, laptop, k3s nodes). Peers dial the VPS endpoint; this makes them portable.

### Addressing plan (example):
* WireGuard subnet: 10.100.0.0/24
* VPS (server) address: 10.100.0.1/24
* Peer #1 : 10.100.0.2/24
* Peer #2 : 10.100.0.3/24

### Security posture: 
* explicitly set Endpoint on peers (If device is portable like a laptop, do not set endpoint explicit
* constrain AllowedIPs per peer
* use a preshared key for an extra layer of secrecy. 

## 2. Install WireGuard

### Standard Update + Wireguard Command
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y wireguard
```
## 3. Generate keys
### On each node (VPS and every peer):
```bash
# Create a safe place for keys
sudo mkdir -p /etc/wireguard/keys
sudo chmod 700 /etc/wireguard/keys
cd /etc/wireguard/keys

# Generate private/public keypair
wg genkey | sudo tee private.key > /dev/null
sudo sh -c 'cat private.key | wg pubkey > public.key'

# (Optional but recommended) One preshared key per peer link
# Run on either side once per pair you connect (VPS<->Peer)
wg genpsk | sudo tee psk-peer1.key > /dev/null

sudo chmod 600 *.key
```
### Collect the public keys and psk you’ll need when writing configs:
```bash 
sudo cat /etc/wireguard/keys/public.key
```
* **VPS**: /etc/wireguard/keys/public.key

* **Peer1**: /etc/wireguard/keys/public.key (from that peer)

* **PSK (per-link)**: /etc/wireguard/keys/psk-peer1.key

## 4. Enable IP forwarding (VPS)
If the VPS will route traffic between peers or to the internet:

### Enable immediately
```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```
### Make persistent
```bash
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-wireguard.conf
sudo sysctl --system
```

**Note**: For the topology of this homelab, IP forwarding is requried. for a full mesh (everyone is peer of everyone) this is not required. Hub and Spoke model used here keeps peers contained and gives more control from the hub. 

## 5. Wireguard Specific Firewall Settings
**Author Note**: With a headless setup, verify the tunnel works and SSH works through the wireguard tunnel before enabling firewall.
* **RISK OF LOCKING YOUR SELF OUT** 

For general firewall information click [here]()
### 1. Allow WireGuard UDP port
```bash
sudo ufw allow 51820/udp
```
### 2. Allow forwarding within the WG subnet
This is helpful when trying to follow least privilege security practices. Only explicitly defined forwarding routes can happen. 
#### Example: Allow Peer1(10.100.0.2) and Peer2(10.100.0.3) to reach 10.100.0.4
```bash
sudo ufw route allow in on wg0 to 10.100.0.4 proto any from 10.100.0.3
sudo ufw route allow in on wg0 to 10.100.0.4 proto any from 10.100.0.4
```
### Alternative: broadly allow WG subnet east-west 
This is **less secure** but easier to navigate. new nodes do not need additional configuration. But compromised machine can access everything.
#### Example: 
```bash
sudo ufw route allow in on wg0 from 10.100.0.0/24 to 10.100.0.0/24

```

## 6) Server config (/etc/wireguard/wg0.conf) on the VPS (Listener)
### Interface
* **wg0.conf**: file name is the name of the wireguard interface. the tunnel is now wg0
* **[Interface]**: defines the local WireGuard network interface on this machine (e.g., wg0). It’s the “self” side of the tunnel
* **Address**: Wireguard address for the machine the config file is on
* **PrivateKey**: the private key on the machine. for VPS machine, Private key is VPS private key
#### Example:
```bash
[Interface]
Address = 10.100.0.1/24
ListenPort = 51820
# VPS private key:
PrivateKey = <contents of /etc/wireguard/keys/private.key>
```
```
# Optional: if you want to push DNS to peers (Linux peers can ignore if undesired)
# DNS = 10.100.0.1

# Peer 1 (Laptop)
[Peer]
# Laptop public key:
PublicKey = <peer1_public_key>
# Extra secrecy:
PresharedKey = <contents of /etc/wireguard/keys/psk-peer1.key>
# Only this IP belongs to Peer1
AllowedIPs = 10.100.0.3/32

# Peer 2 (Devhost)
[Peer]
PublicKey = <peer2_public_key>
PresharedKey = <contents of /etc/wireguard/keys/psk-peer2.key>
AllowedIPs = 10.100.0.6/32
```

Notes

The server does not set Endpoint for peers; peers dial in to the server.

Use tight AllowedIPs on server peers (one /32 per device) for clean routing and access control.

7) Peer config (/etc/wireguard/wg0.conf) on a peer (e.g., Laptop)
[Interface]
Address = 10.100.0.3/24
PrivateKey = <peer1_private_key>
# Optional local DNS while tunnel is up
# DNS = 10.100.0.1

[Peer]
# VPS public key:
PublicKey = <vps_public_key>
# Same PSK used on the VPS for this link:
PresharedKey = <contents of /etc/wireguard/keys/psk-peer1.key>
# Route only the WG subnet through the tunnel (zero trust style)
AllowedIPs = 10.100.0.0/24
# VPS endpoint (public IP or DNS + UDP port)
Endpoint = <vps.public.ip.or.dns>:51820
# Keep NATs alive when peer is behind coffee shop/home routers
PersistentKeepalive = 25


Portability:

By setting Endpoint on the peer, the peer can roam between networks and still reconnect to the VPS.

Do not set an Endpoint on the VPS’s [Peer] entries; the VPS waits for the peers.

8) Bring the tunnel up

On each node:

# Check config ownership/permissions (wg requires 600)
sudo chmod 600 /etc/wireguard/wg0.conf

# Start once
sudo wg-quick up wg0

# Enable at boot
sudo systemctl enable wg-quick@wg0

# Show status / live view
sudo wg show
ip addr show wg0


Test basic reachability:

# From peer to VPS
ping 10.100.0.1

# From VPS to peer
ping 10.100.0.3

9) (Optional) Add more peers

Repeat keygen on the new peer, create a PSK for that pair, add a [Peer] block on the VPS, and configure the peer with VPS’s public key + endpoint. Use unique /32 AllowedIPs per peer on the server.

10) Hardening tips

Use preshared keys (PresharedKey) for every VPS↔Peer pair. It adds secrecy even if a public key is compromised.

Tight AllowedIPs:

On the server, use /32 per peer.

On peers, route only what you need (e.g., 10.100.0.0/24 or specific hosts like 10.100.0.7/32).

Explicit endpoints: for peers, always set Endpoint = vps.example.com:51820.

UFW forward rules: allow only what you intend across wg0.

Key hygiene: rotate keys periodically; store under /etc/wireguard/keys with 600 perms.

11) Quick troubleshooting
# See handshakes, endpoints, latest handshake time, transfer stats
sudo wg show

# Interface up?
ip link show wg0

# Routing?
ip route show table main | grep 10.100.0.

# Firewall counters (are forwards being dropped?)
sudo iptables -vnL FORWARD
sudo ufw status verbose

# Logs
journalctl -u wg-quick@wg0 -e

12) Minimal copy/paste cheat sheet

VPS

sudo apt install -y wireguard
sudo mkdir -p /etc/wireguard/keys && sudo chmod 700 /etc/wireguard/keys
cd /etc/wireguard/keys
wg genkey | sudo tee private.key >/dev/null
sudo sh -c 'cat private.key | wg pubkey > public.key'
wg genpsk | sudo tee psk-peer1.key >/dev/null
sudo chmod 600 *.key
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-wireguard.conf
sudo sysctl --system
sudo ufw allow 51820/udp


Peer

sudo apt install -y wireguard
sudo mkdir -p /etc/wireguard/keys && sudo chmod 700 /etc/wireguard/keys
cd /etc/wireguard/keys
wg genkey | sudo tee private.key >/dev/null
sudo sh -c 'cat private.key | wg pubkey > public.key'
wg genpsk | sudo tee psk-peer1.key >/dev/null
sudo chmod 600 *.key


Then write the two wg0.conf files as shown above, sudo wg-quick up wg0 on both sides, and you’re live.