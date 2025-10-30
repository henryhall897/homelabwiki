---
title: MoCA Adapters (Deprecated Experiment)
description: Why MoCA are suboptimal for a home lab setup
published: true
date: 2025-10-30T14:25:40.742Z
tags: public, infrastructure, hardware, moca
editor: markdown
dateCreated: 2025-10-19T17:03:34.924Z
---

# MoCA Adapters (Deprecated Experiment)
## Overview

MoCA (Multimedia over Coax Alliance) adapters are designed to transmit Ethernet data over existing coaxial cabling.
While useful in homes without Ethernet wiring, they are not well suited for high-performance or multi-device network backbones.

## Why MoCA Is Suboptimal

### Shared-Loop Limitation
Coaxial cabling operates as a **shared bus** — only one device can effectively transmit on the loop at a time.
When multiple adapters are connected, they must wait their turn to send data, introducing contention and latency.
This design severely limits throughput and stability compared to full-duplex Ethernet.

### Packet Loss Under Load
Because of the shared medium, simultaneous data transfers result in collisions and retransmissions.
The result is **measurable packet loss and jitter**, especially noticeable during streaming, SSH sessions, or gaming.

### Interference and Distance Loss
Signal quality degrades quickly across splitters, long coax runs, or older in-wall cabling.
MoCA performs best only with clean, short cable paths and minimal branching.

### Troubleshooting Complexity
Unlike Ethernet, coax networks can hide loops and signal reflections that are difficult to diagnose without specialized tools.

### Visual Example


## When It Might Still Be Acceptable
A single-device bridge (e.g., connecting a smart TV or one PC to the router) can work reliably if the coax path is short and isolated.

However, using MoCA as the primary network backbone or multi-node link leads to **performance issues and instability**.

## Personal Note

Originally, the home network used MoCA adapters to link the downstairs modem to the upstairs router via coax.
While functional, the connection suffered from severe packet loss and unpredictable latency.
Replacing the MoCA link with direct Ethernet cabling up the stairs resulted in dramatically improved stability and throughput — confirming that MoCA was a poor fit for core infrastructure