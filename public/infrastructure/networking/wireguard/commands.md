---
title: Wireguard Commands
description: A list of all Wireguard commands
published: true
date: 2025-10-30T15:08:18.219Z
tags: public, infrastructure, networking, wireguard, commands
editor: markdown
dateCreated: 2025-10-19T17:05:39.959Z
---

# WireGuard Commands (Ubuntu) — Cheat Sheet
[Overview](/public/infrastructure/networking/wireguard) - [Setup](/public/infrastructure/networking/wireguard/setup) - [Troubleshooting](/public/infrastructure/networking/wireguard/troubleshooting) - [Commands](/public/infrastructure/networking/wireguard/commands)

---

Works with Ubuntu 22.04+. Replace wg0 with your interface name.

## Install & files
```bash
sudo apt update && sudo apt install -y wireguard
ls -l /etc/wireguard/               # configs live here (wg0.conf)
sudo chmod 600 /etc/wireguard/wg0.conf
```
## Bring interfaces up/down & autostart
```bash
sudo wg-quick up wg0
sudo wg-quick down wg0
sudo systemctl enable wg-quick@wg0
sudo systemctl disable wg-quick@wg0
sudo systemctl restart wg-quick@wg0
```

## Show status / live info
```bash
sudo wg show                       # peers, endpoints, handshakes, transfer
sudo wg show wg0                   # single interface
ip addr show wg0                   # interface + IP
ip route show | grep 10.100.0.     # routes for your WG subnet
```

## Keys (generate safely)
### In a protected directory
```bash
sudo mkdir -p /etc/wireguard/keys && sudo chmod 700 /etc/wireguard/keys
cd /etc/wireguard/keys
wg genkey | sudo tee private.key >/dev/null
sudo sh -c 'cat private.key | wg pubkey > public.key'
wg genpsk | sudo tee psk-peerX.key >/dev/null
sudo chmod 600 *.key
```
## Add/modify peers (on a running interface)
Persistent changes belong in /etc/wireguard/wg0.conf; these are temporary runtime edits.

### Add a peer (runtime)
```bash
sudo wg set wg0 peer <PUBKEY> allowed-ips 10.100.0.2/32
```
### Add/update peer with endpoint and PSK
```bash
sudo wg set wg0 peer <PUBKEY> preshared-key /etc/wireguard/keys/psk-peer2.key \
  endpoint vps.example.com:51820 allowed-ips 10.100.0.2/32
```
### Remove a peer (runtime)
```bash
sudo wg set wg0 peer <PUBKEY> remove
```

## Troubleshooting quickies
```bash
journalctl -u wg-quick@wg0 -e       # service logs
sudo ss -lunp | grep 51820          # listening UDP port
sudo ufw status verbose             # firewall summary
sudo iptables -S; sudo iptables -t nat -S   # iptables rules (if used)
sudo nft list ruleset               # nftables rules (if used)
sudo tcpdump -ni eth0 udp port 51820 # see handshakes hitting VPS
sudo tcpdump -ni wg0                # see WG traffic on the tunnel
```
## Forwarding & NAT (hub)
### Enable forwarding
```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-wireguard.conf
sudo sysctl --system
```
### MASQUERADE WG subnet out public IF (change eth0 if needed)
```bash
sudo iptables -t nat -A POSTROUTING -s 10.100.0.0/24 -o eth0 -j MASQUERADE
```
### DNS (optional, wg-quick convenience)
#### In [Interface] on a client:
```bash
DNS = 10.100.0.1     # use resolver over WG while wg0 is up
```
## Mobile (qr for Android/iOS clients)
```bash
sudo apt install -y qrencode
sudo qrencode -t ansiutf8 < /etc/wireguard/wg0.conf
``` 
## Wireguard Articles
* [Overview](/public/infrastructure/networking/wireguard)
* [Setup](/public/infrastructure/networking/wireguard/setup)
* [Troubleshooting](/public/infrastructure/networking/wireguard/troubleshooting)
* [Commands](/public/infrastructure/networking/wireguard/commands)