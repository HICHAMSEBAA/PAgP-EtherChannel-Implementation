# Enterprise PAgP EtherChannel Implementation, Load Balancing Optimization & DTP Hardening

![Platform](https://img.shields.io/badge/Platform-EVE--NG-blue)
![Vendor](https://img.shields.io/badge/Vendor-Cisco%20IOS-orange)
![Protocol](https://img.shields.io/badge/Protocol-PAgP%20%7C%20EtherChannel-green)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

This repository contains the complete network architecture, Cisco IOS configurations, Linux host parameters, and verification logs for a High Availability Port Aggregation Protocol (PAgP) EtherChannel lab integrated with ASIC Load-Balancing Tuning and DTP Suppression.

---

## 📋 Overview & Architecture

The objective of this design is to optimize bandwidth utilization and enforce Layer 2 resilience across redundant inter-switch links while securing the control plane against unauthorized trunk negotiation:

* **Link Aggregation & Resiliency:** Switch1 and Switch2 bundle four physical links (`GigabitEthernet0/0 - 0/3`) into a single logical `Port-channel 1` using Cisco PAgP (`desirable`/`auto` modes), multiplying inter-switch capacity to 4 Gbps while preventing Spanning Tree Protocol (STP) link blocking.
* **Bidirectional Loop Protection (Non-Silent Mode):** All member ports enforce Non-Silent mode (`non-silent`), requiring active two-way PAgP control frame exchange before bringing links online to prevent unidirectional traffic loops.
* **ASIC Load-Balancing Optimization:** To prevent traffic polarization from the single Linux Ubuntu Server (`192.168.1.10`), Switch1 is configured with destination IP hashing (`port-channel load-balance dst-ip`). This evenly distributes outbound server flows across all four physical trunk members.
* **Layer 2 Control Plane Hardening:** Dynamic Trunking Protocol (DTP) frame generation is explicitly disabled (`switchport nonegotiate`) across physical and logical trunk interfaces to enforce static 802.1Q encapsulation and mitigate VLAN hopping risks.

---

## 📐 Network Topology

```text
                     +--------------------------+
                     |   Linux Ubuntu Server    |
                     |     192.168.1.10/24      |
                     +------------+-------------+
                                  |
                                  | (e0 / Gi1/0)
                                  |
                            +-----+-----+
                            |  Switch1  | (Load-Balance: dst-ip)
                            +--+--+--+--+
                               |  |  |  |
         (Gi0/0) --------------+  |  |  +-------------- (Gi0/3)
         (Gi0/1) -----------------+  +----------------- (Gi0/2)
                               |  |  |  |
                            +--+--+--+--+  [PAgP Po1 - 4x1Gbps Trunk]
                            |  Switch2  |  (VLAN 10: 192.168.1.0/24)
                            +--+--+--+--+
                              /   |   \
               (Gi1/0)       /    |    \       (Gi1/2)
              +-------------+     |     +-------------+
              |                   | (Gi1/1)           |
      +-------+-------+   +-------+-------+   +-------+-------+
      |  Debian PC1   |   |  Debian PC2   |   |  Debian PC3   |
      | 192.168.1.11  |   | 192.168.1.12  |   | 192.168.1.13  |
      +---------------+   +---------------+   +---------------+
