---
title: Wireguard Setup
description: How to do initial wireguard setup
published: true
date: 2025-10-30T15:06:21.691Z
tags: networking, wireguard, setup
editor: markdown
dateCreated: 2025-10-19T17:05:54.496Z
---

# WireGuard Setup (Ubuntu)
[Overview](/public/infrastructure/networking/wireguard) - [Setup](/public/infrastructure/networking/wireguard/setup) - [Troubleshooting](/public/infrastructure/networking/wireguard/troubleshooting) - [Commands](/public/infrastructure/networking/wireguard/commands)

---

**Scope**: Ubuntu 22.04+ on a VPS (listener/server) and one or more peers (laptops, home nodes, etc.).
**Goal**: install WireGuard, generate keys (including an optional preshared key), pick your listener, and bring up your first tunnel with tight endpoint + routing controls.

## 1. Decide your topology
* **Listener (Server)**: your VPS should listen on a stable public IP/DNS and port 51820/udp (or any unused UDP port).
* **Peers**: everything else (home dev box, laptop, k3s nodes). Peers dial the VPS endpoint; this makes them portable.
### Addressing plan (example):
* **WireGuard subnet**: 10.100.0.0/24
* **Peer 1 (VPS)**: 10.100.0.1/32
* **Peer 2 (Server Spoke)** : 10.100.0.2/32
* **Peer 3 (User Interface)** : 10.100.0.3/24
### Security posture: 
* explicitly set Endpoint on peers (If device is portable like a laptop, do not set endpoint explicit
* constrain AllowedIPs per peer
* use a preshared key for an extra layer of secrecy. 
### Example Topology Diagram
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

For general firewall information click [here](../firewall)
### 1. Allow WireGuard UDP port
```bash
sudo ufw allow 51820/udp
```
#### To be more strict, it is possible to require from certain ips (like your specific public IP)
```bash
sudo ufw allow from <your_public_ip> to any port 51820 proto udp
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

## 6. Server config (/etc/wireguard/wg0.conf) on the VPS (Listener)
### Interface
* **wg0.conf**: file name is the name of the wireguard interface. the tunnel is now wg0
* **[Interface]**: defines the local WireGuard network interface on this machine (e.g., wg0). It’s the “self” side of the tunnel
* **Address**: Wireguard address for the machine the config file is on
* **PrivateKey**: the private key on the machine. for VPS machine, Private key is VPS private key
#### Example: [Interface]
```bash
[Interface]
Address = 10.100.0.1/24
ListenPort = 51820
PrivateKey = <contents of /etc/wireguard/keys/private.key>
```
### Peers
* **PublicKey**: The public key of the device on the other side of the tunnel.
* **PresharedKey**: Symmetrical Key unqiue to each peer, each side gets same key 
* **AllowedIPs**: Signifies which wireguard address are allowed to use the tunnel between to contact the node. 
	* **Forwarded IPs**: Need to allow multiple IPs from the peer performing the forwarding
* **PersistentKeepalive**: Tells Wireguard to actively send packets and ensure connection still exists through consistent handshakes.
	* good for roaming devices like a laptop taken to different networks or for troublshooting
	* number corresponds to seconds (25 = 25 seconds)
#### Example: Peer 2
* Has a tunnel (Direct peer) with **Peer 1**
* Also needs to accept traffic from **Peer 3** (indirect peer) that is forwarded from **Peer 1**
#### Peer 2 wg0.conf peer section
```bash
# tunnel to peer 1
[Peer]
PublicKey = <peer1_public_key>
PresharedKey = <contents of /etc/wireguard/keys/psk-peer1.key>
# allows both peer 1 and peer 3
AllowedIPs = 10.100.0.1/32, 10.100.0.3/32
PersistentKeepalive = 25 
```
### Notes
* The server does not set Endpoint for peers; peers dial in to the server.
* Use tight AllowedIPs on server peers (one /32 per device) for clean routing and access control.
	* Can add extra AllowedIPs for forwarded address
 * PersistentKeepalive can be omitted on the Hub
 
### Full Example Config Files
Following diagram example above:
#### Hub (VPS) - listener and hub
```bash
[Interface]
Address = 10.100.0.1/32
PrivateKey = <peer1_private_key>
ListenPort = 51280

# Peer 2 server node
[Peer]
PublicKey = <peer 2 pub key>
PresharedKey = <peer 2 to vps preshared key (psk)>
AllowedIPs = 10.100.0.2/32

[Peer]
PublicKey = <peer 3 pub key>
PresharedKey = <peer 3 to vps psk>
AllowedIPs = 10.100.0.3/32
```

#### Peer 2 (Server Spoke)
```bash
[Interface]
Address = 10.100.0.2/32
PrivateKey = <peer2_private_key>
ListenPort = 51820

# Peer 1 Hub
[Peer]
PublicKey = <peer 1 pub key>
PresharedKey = <peer 2 to vps preshared key (psk)>
# Both peer 1 and peer 3 can use peer 1's tunnel
AllowedIPs = 10.100.0.1/32, 10.100.0.3/32
Endpoint = hubip:51820
PersistentKeepalive = 25
```
#### Peer 3 (User Interface Spoke):
```bash
[Interface]
Address = 10.100.0.3/32
PrivateKey = <peer 3 private key>
ListenPort = 51820

# Peer 1 Hub
[Peer]
PublicKey = <peer 1 pub key>
PresharedKey = <peer 3 to vps preshared key (psk)>
# Peer 3 can specify all nodes or allow everything to make adding new nodes easier.
# here, all nodes are allowed for convenience of wireguard on the user interface side. 
# just need to configure spokes to allow the user interface
AllowedIPs = 10.100.0.0/24
Endpoint = hubip:51820
PersistentKeepalive = 25
```

## 8. Bring the tunnel up
**On each node**:
### Check config ownership/permissions (wg requires 600)
```bash
sudo chmod 600 /etc/wireguard/wg0.conf
```
### Start once
```bash
sudo wg-quick up wg0
```
### Enable at boot
```bash
sudo systemctl enable wg-quick@wg0
```
### Show status / live view
```bash
sudo wg show
ip addr show wg0
```

### Test basic reachability:
#### From peer to VPS
```bash
ping 10.100.0.1
```
#### From VPS to peer
```bash
ping 10.100.0.3
```
## 9. (Optional) Add more peers

Repeat keygen on the new peer, create a PSK for that pair, add a [Peer] block on the VPS, and configure the peer with VPS’s public key + endpoint. Use unique /32 AllowedIPs per peer on the server.

## 10. Hardening tips
* Use preshared keys (PresharedKey) for every VPS↔Peer pair. It adds secrecy even if a public key is compromised.
* Tight AllowedIPs:
* On the server, use /32 per peer.
* On peers, route only what you need (e.g., 10.100.0.0/24 or specific hosts like 10.100.0.7/32).
* Explicit endpoints: for peers, always set Endpoint = vps.example.com:51820.
* UFW forward rules: allow only what you intend across wg0.
* Key hygiene: rotate keys periodically; store under /etc/wireguard/keys with 600 perms.

## Optional: if you want to push DNS to peers (Linux peers can ignore if undesired)
**DNS**: Setting a DNS in the interface means that the node will use the tunnel for DNS resolution
### Pros
* **Privacy**: Sends DNS queries through the tunnel (to your VPN resolver), avoiding ISP DNS.
* **Leak reduction**: Paired with AllowedIPs that include the DNS server, helps prevent DNS leaking outside the tunnel.
### Cons
* **Resolver dependency**: If the WG DNS (e.g., 10.100.0.1) is down or unreachable, lookups fail.
* **Route coupling**: You must include the DNS IP in AllowedIPs; otherwise DNS packets won’t reach it.
* **Linux variance**: Behavior depends on systemd-resolved/resolvconf/NetworkManager; on some setups it may be ignored or fight with other managers.
* **Captive portals / public Wi-Fi**: Forcing VPN DNS can break initial captive portal access until you connect and authenticate.
* **All-or-nothing**: Basic DNS= sets a global resolver; it doesn’t do per-domain split-DNS by itself (needs extra resolver config).
* **Troubleshooting complexity**: If name resolution breaks, it’s one more moving part (WG + resolver + routes).
##### TLDR: 
* The Homelab setup does not use wireguard DNS. That would make the home network work more like services such as NORD VPN 
* Would cause more use on VPS. could lead to bottlenecks or extra charges

## Wireguard Articles
* [Overview](/public/infrastructure/networking/wireguard)
* [Setup](/public/infrastructure/networking/wireguard/setup)
* [Troubleshooting](/public/infrastructure/networking/wireguard/troubleshooting)
* [Commands](/public/infrastructure/networking/wireguard/commands)