---
title: MoCA Adapters (Deprecated Experiment)
description: Why MoCA are suboptimal for a home lab setup
published: true
date: 2025-10-30T14:24:34.165Z
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
```mermaid
flowchart LR
    subgraph MoCA_Network["🏠 MoCA Shared Coax Network (Suboptimal)"]
        direction LR
        R1["Router/Modem"]
        L1["Splitter"]
        A1["MoCA Adapter 1<br>(Living Room)"]
        A2["MoCA Adapter 2<br>(Office PC)"]
        A3["MoCA Adapter 3<br>(Upstairs TV)"]
        R1 --- L1
        L1 --- A1
        L1 --- A2
        L1 --- A3
    end

    subgraph Ethernet_Network["🏠 Ethernet Switched Network (Optimal)"]
        direction LR
        S1["Network Switch"]
        E1["PC"]
        E2["Server"]
        E3["Smart TV"]
        S1 --- E1
        S1 --- E2
        S1 --- E3
    end

    classDef shared fill:#8b0000,stroke:#ff5555,stroke-width:2px,color:#fff
    classDef optimal fill:#004d00,stroke:#00cc00,stroke-width:2px,color:#fff
    class MoCA_Network shared
    class Ethernet_Network optimal

    note right of MoCA_Network::bottom
      "Shared coax bus → contention, latency, packet loss"
    end

    note right of Ethernet_Network::bottom
      "Full-duplex links → stable, high-throughput communication"
    end
```

## When It Might Still Be Acceptable
A single-device bridge (e.g., connecting a smart TV or one PC to the router) can work reliably if the coax path is short and isolated.

However, using MoCA as the primary network backbone or multi-node link leads to **performance issues and instability**.

## Personal Note

Originally, the home network used MoCA adapters to link the downstairs modem to the upstairs router via coax.
While functional, the connection suffered from severe packet loss and unpredictable latency.
Replacing the MoCA link with direct Ethernet cabling up the stairs resulted in dramatically improved stability and throughput — confirming that MoCA was a poor fit for core infrastructure