---
title: Wireguard Troubleshooting
description: Common Issues with Wireguard and how to solve them
published: true
date: 2025-10-10T22:45:37.972Z
tags: wireguard, networking, troubleshooting, infrastructure, public
editor: markdown
dateCreated: 2025-10-10T22:45:37.971Z
---

# WireGuard Troubleshooting (Ubuntu) — Handshake + Routing

## Scope: 
For when peers won’t handshake or traffic won’t pass. Works for hub-and-spoke and mesh(VPS hub: 10.100.0.1, spokes like 10.100.0.2, 10.100.0.3).

## Quick “Why it fails” cheatsheet
* **Port/endpoint**: Server UDP port closed, wrong IP/DNS, or cloud firewall/security group blocking.
* **Keys**: Private/Public swapped or stale; PSK mismatch.
* **AllowedIPs**: Missing the other peer’s /32, overlaps/duplicates, or routes hijacked.
* **NAT/roaming**: Peer behind NAT needs PersistentKeepalive = 25.
* **Forwarding/firewall**: ip_forward off on hub; UFW/nftables/SGs deny forwarding.
* **Permissions**: wg0.conf or private.key not chmod 600.
* **Conf file misnaming**: is named wrong wg0.conf matches wg0.conf not wg120.conf on a different machine
* **Interface**: Wrong interface names in firewall/NAT rules; MTU issues causing stalls.
* **DNS (for Endpoint)**: Endpoint hostname doesn’t resolve (or resolves to IPv6 your path doesn’t carry).

## 1. See what WireGuard thinks (both sides)
```bash
sudo wg show
```
### Look for:
* public key (matches the other side’s [Peer]?)
* endpoint (VPS: blank for spokes, Spoke: vps:51820)
* latest handshake (if “never”, it isn’t reaching)
* transfer counters (bytes ↑/↓)
* allowed ips (does each side list the other’s /32?)

#### If handshake never happens:
* Server isn’t reachable (UDP blocked / wrong endpoint), or
* Keys/PSK don’t match, or
* Peer not sending (no traffic + no keepalive).

## 2. Verify the hub (VPS) is reachable
### On the VPS:

#### Is wg0 up?
```bash
ip a show wg0
```
#### Listening on the right UDP port?
```bash
sudo ss -lunp | grep 51820
# or, older:
sudo netstat -lunp | grep 51820
```
### From outside (a third box) test the UDP port:
#### nmap will say "open|filtered" for UDP usually, but closed means trouble
```bash
nmap -sU -p 51820 <VPS_PUBLIC_IP>
```

### Cloud firewall / security group:
* Ensure UDP 51820 is allowed inbound to the VPS.
* Ensure outbound UDP is not blocked (some hosts restrict egress).

### On the VPS host firewall:
```bash
sudo ufw status verbose
```
#### Expecting: 
51820/udp ALLOW IN

## 3. Keys & PSK sanity (both sides)
### On each node:
```bash
sudo ls -l /etc/wireguard/keys
sudo chmod 600 /etc/wireguard/keys/private.key
sudo chmod 600 /etc/wireguard/wg0.conf
```
* **Public/Private pairing**: cat private.key | wg pubkey should match the saved public.key.
* Don’t paste file paths into PrivateKey; paste the contents (base64).
* PSK must be identical on both ends of a pair; don’t reuse the wrong PSK.

## 4. Endpoint, NAT, and keepalive (spokes)

### On roaming / NATted peers:
```bash
[Peer]
Endpoint = vps.example.com:51820
PersistentKeepalive = 25
```

* PersistentKeepalive = 25 keeps NAT pinholes open.
* If DNS might fail, try the VPS public IP directly.
* If your VPS has IPv6 DNS but no IPv6 path, use an IPv4 A record or the raw IPv4.

## 5) AllowedIPs ≡ routing (most common mistake)

Hub (VPS) should have one /32 per spoke:

### On VPS
```bash
[Peer] # Peer 2
PublicKey = <peer2_pub>
AllowedIPs = 10.100.0.2/32

[Peer] # Peer 3
PublicKey = <peer3_pub>
AllowedIPs = 10.100.0.3/32
```

Spokes should target the hub and any other spoke they want to reach via the hub:

### On Peer 2 (server spoke)
```bash
[Peer] # Hub
PublicKey = <vps_pub>
AllowedIPs = 10.100.0.1/32, 10.100.0.3/32   # hub + user spoke via hub
Endpoint = vps.example.com:51820
PersistentKeepalive = 25
```
### On Peer 3 (user spoke)
```bash
[Peer] # Hub
PublicKey = <vps_pub>
AllowedIPs = 10.100.0.1/32   # hub
Endpoint = vps.example.com:51820
PersistentKeepalive = 25
```

## Gotchas
* Do not give multiple spokes the same interface Address (unique /32 per peer).
* Avoid overlapping AllowedIPs on the same machine (routing confusion).
* If you use AllowedIPs = 0.0.0.0/0 on a spoke, remember all traffic goes to the hub—often not desired.

### Show routes:
```bash
ip route show table main | grep 10.100.
```
## 6. Forwarding and firewall (hub)
### Hub must forward packets between spokes:
#### Forwarding enabled?
```bash
cat /proc/sys/net/ipv4/ip_forward   # should be 1
```
#### Make persistent if not:
```bash
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-wireguard.conf
sudo sysctl --system
```

### UFW/nftables must allow FORWARD between wg0 and itself (and/or to the Internet if NATing):

#### UFW examples (east-west)
```bash
sudo ufw route allow in on wg0 from 10.100.0.0/24 to 10.100.0.0/24
# or tight per-host forwards, e.g. only 10.100.0.3 -> 10.100.0.2
sudo ufw route allow in on wg0 from 10.100.0.3 to 10.100.0.2
```

### If exposing a service publicly through the hub:
#### Ensure DNAT/MASQUERADE rules are correct and interface names match (e.g., eth0, wg0).
##### List rules:
```bash
sudo iptables -S
sudo iptables -t nat -S
# or nftables:
sudo nft list ruleset
```
## 7. MTU and “it handshakes but stalls”
**Symptoms**: pings work small, big packets hang; some protocols time out.

Try lowering **MTU** on the spoke:
```bash
[Interface]
MTU = 1420
```
Then:
```bash
sudo wg-quick down wg0 && sudo wg-quick up wg0
```
## 8. Logs & packet capture
### System logs:
```bash
journalctl -u wg-quick@wg0 -e
```
### See UDP 51820 arriving on the VPS:
```bash
sudo tcpdump -ni eth0 udp port 51820
```

### See WG traffic on the interface:
```bash
sudo tcpdump -ni wg0
```

If nothing hits the VPS public NIC, you have an upstream firewall/NAT issue or wrong Endpoint.

## 9. Minimal known-good configs (sanity check)

### VPS (hub)
```bash
[Interface]
Address = 10.100.0.1/24
ListenPort = 51820
PrivateKey = <vps_priv>

[Peer] # Peer 2
PublicKey = <peer2_pub>
PresharedKey = <vps-peer2_psk>
AllowedIPs = 10.100.0.2/32

[Peer] # Peer 3
PublicKey = <peer3_pub>
PresharedKey = <vps-peer3_psk>
AllowedIPs = 10.100.0.3/32
```

### Peer 2 (server spoke)
```bash
[Interface]
Address = 10.100.0.2/32
PrivateKey = <peer2_priv>

[Peer] # Hub
PublicKey = <vps_pub>
PresharedKey = <vps-peer2_psk>
AllowedIPs = 10.100.0.1/32, 10.100.0.3/32
Endpoint = <vps_public_ip_or_dns>:51820
PersistentKeepalive = 25
```

### Peer 3 (user spoke)
```bash
[Interface]
Address = 10.100.0.3/32
PrivateKey = <peer3_priv>

[Peer] # Hub
PublicKey = <vps_pub>
PresharedKey = <vps-peer3_psk>
AllowedIPs = 10.100.0.1/32, 10.100.0.2/32
Endpoint = <vps_public_ip_or_dns>:51820
PersistentKeepalive = 25
```
## 10. Common “gotchas” checklist
* Wrong port or protocol (must be UDP on the VPS).
* Cloud SG / provider firewall blocking UDP 51820.
* VPS host firewall blocking input or forward.
* Endpoint wrong (typo, wrong DNS record, IPv6 mismatch).
* PrivateKey/PublicKey swapped, or PSK mismatch.
* AllowedIPs missing the other peer’s /32, or conflicting overlaps.
* Duplicate WG Address on two peers.
* No PersistentKeepalive on a NATted/roaming peer.
* ip_forward=0 on the hub.
* Interface names wrong in NAT/forward rules (ens3 vs eth0).
* MTU too high on paths with overhead (try 1420).
* File perms too open; wg-quick/kernel refuses to load (chmod 600).
* DNS set in [Interface] but unreachable per AllowedIPs.