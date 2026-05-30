# 🏥 City Smart Hospital Network Design

A fully simulated enterprise hospital network built in **Cisco Packet Tracer** as part of the Computer Networks course (Spring 2026). The project implements a multi-zone, VLAN-segmented network with OSPF dynamic routing, NAT, DHCP, DNS, and failover capabilities.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Network Architecture](#network-architecture)
- [Addressing Scheme](#addressing-scheme)
- [Features Implemented](#features-implemented)
- [Failover & Redundancy](#failover--redundancy)
- [Protocol Analysis](#protocol-analysis)
- [Tools & Technologies](#tools--technologies)
- [How to Run](#how-to-run)

---

## Project Overview

The City Smart Hospital Network models a real-world hospital infrastructure divided into multiple functional zones:

- **Administration Building** — Patient Records, Finance, HR
- **Medical Wards** — General Ward, ICU
- **Outpatient Clinic** — Consultation Rooms, Guest Wi-Fi
- **Data Center** — DNS Server, Web Server
- **Remote Radiology Clinic** — Phase 2 expansion

The network uses private IP addressing from the `172.16.0.0/16` block and connects to a simulated ISP for external internet access.

---

## Network Architecture

```
[ Admin Zone ]──────────────────────────────────┐
  VLAN 10: Patient Records (172.16.10.0/24)      │
  VLAN 20: Finance         (172.16.20.0/24)      │
  VLAN 30: HR              (172.16.30.0/24)      │
                                                  │
[ Medical Zone ]                                  │
  VLAN 40: General Ward    (172.16.40.0/24)      │
  VLAN 50: ICU             (172.16.50.0/24)      ├──► Core Data R ──► ISP R ──► Internet
                                                  │
[ Outpatient Zone ]                               │
  VLAN 60: Consultation    (172.16.60.0/24)      │
  VLAN 70: Guest Wi-Fi     (172.16.70.0/24)      │
                                                  │
[ Data Center ]                                   │
  VLAN 80: Servers         (172.16.80.0/24)      │
                                                  │
[ Remote Radiology Clinic ]─────────────────────┘
  VLAN 90: Radiology        (172.16.90.0/24)
```

Inter-VLAN routing is handled via **Router-on-a-Stick** sub-interfaces on zone routers. All routers run **OSPF Area 0** for dynamic route propagation.

---

## Addressing Scheme

### VLAN Subnets (FLSM /24)

| Zone / Department | VLAN | Network | Default Gateway |
|---|---|---|---|
| Admin: Patient Records | 10 | 172.16.10.0/24 | 172.16.10.1 |
| Admin: Finance | 20 | 172.16.20.0/24 | 172.16.20.1 |
| Admin: HR | 30 | 172.16.30.0/24 | 172.16.30.1 |
| Medical: General Ward | 40 | 172.16.40.0/24 | 172.16.40.1 |
| Medical: ICU | 50 | 172.16.50.0/24 | 172.16.50.1 |
| Outpatient: Consultation | 60 | 172.16.60.0/24 | 172.16.60.1 |
| Outpatient: Guest Wi-Fi | 70 | 172.16.70.0/24 | 172.16.70.1 |
| Data Center: Servers | 80 | 172.16.80.0/24 | 172.16.80.1 |
| Remote Radiology Clinic | 90 | 172.16.90.0/24 | 172.16.90.1 |

### Point-to-Point Router Links

| Link | Network | Router A IP | Router B IP |
|---|---|---|---|
| Admin R – Core Data R | 172.16.100.0/24 | 172.16.100.1 | 172.16.100.2 |
| Medical R – Core Data R | 172.16.101.0/24 | 172.16.101.1 | 172.16.101.2 |
| Outpatient R – Core Data R | 172.16.102.0/24 | 172.16.102.1 | 172.16.102.2 |
| Medical R – Admin R | 172.16.103.0/24 | 172.16.103.1 | 172.16.103.2 |
| Radiology R – Admin R | 172.16.104.0/24 | 172.16.104.1 | 172.16.104.2 |
| Core Data R – ISP R | 203.0.113.0/30 | 203.0.113.1 | 203.0.113.2 |

### Static Server IPs

| Server | VLAN | IP | Purpose |
|---|---|---|---|
| DNS Server | 80 | 172.16.80.2 | Resolves `www.cityhospital.local` |
| Web Server | 80 | 172.16.80.3 | Hosts the hospital intranet |
| External Web Server | — | 8.8.8.8 | Simulates `www.google.com` |

---

## Features Implemented

- **VLANs** — 9 VLANs separating departments into isolated broadcast domains
- **Trunk Links** — 802.1Q trunking between switches and routers
- **Router-on-a-Stick** — Sub-interfaces for inter-VLAN routing on zone routers
- **DHCP** — Dynamic IP assignment configured on each zone router for connected VLANs
- **OSPF Area 0** — Dynamic routing across all internal routers
- **NAT Overload (PAT)** — Configured on Core Data R to allow internal devices to reach the external network
- **DNS** — Internal DNS server resolving `www.cityhospital.local`
- **Web Server** — Hosts the hospital intranet accessible from any internal device
- **Wired + Wireless Access** — Consultation rooms use wired Ethernet (VLAN 60); Guest Wi-Fi uses an access point (VLAN 70)
- **Remote Clinic Expansion (Phase 2)** — Radiology Clinic added as a new zone connected through Admin R

---

## Failover & Redundancy

A backup link between **Medical R** and **Admin R** (`172.16.103.0/24`) provides path diversity.

**Normal path** (Medical → Data Center):
```
172.16.50.1 → 172.16.101.1 → 172.16.80.3
```

**After primary link failure** (OSPF reconverges automatically):
```
172.16.50.1 → 172.16.103.1 → 172.16.100.1 → 172.16.80.3
```

OSPF detects the topology change via link-state updates and recalculates the shortest available path through Admin R, maintaining full connectivity.

---

## Protocol Analysis

| Protocol | Transport | Port | Reason |
|---|---|---|---|
| DNS | UDP | 53 | Lightweight, fast request-response; avoids TCP handshake overhead |
| HTTP | TCP | 80 | Reliable delivery required; TCP provides acknowledgments, ordering, and retransmission |

Captured using **Packet Tracer Simulation Mode** while accessing `http://www.cityhospital.local` from an edge PC.

---

## Tools & Technologies

- **Cisco Packet Tracer** — Network simulation
- **OSPF** — Dynamic routing protocol (Area 0)
- **FLSM /24 subnetting** — Fixed-length subnet masking from `172.16.0.0/16`
- **VLANs & 802.1Q trunking**
- **DHCP, DNS, HTTP** — Application layer services
- **NAT Overload (PAT)** — Internet access for internal devices

---

## How to Run

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (version 8.x recommended).
2. Clone or download this repository.
3. Open the `.pkt` file in Packet Tracer.
4. Switch to **Simulation Mode** to observe packet flow.
5. To test intranet access, open any PC's Web Browser and navigate to `http://www.cityhospital.local`.
6. To test failover, delete the cable between **Medical R** and **Core Data R** and run a traceroute from ICU 1 to `172.16.80.3`.

---

> **Course:** Computer Networks — Spring 2026  
> **Institution:** Zewail City of Science and Technology
