---
title: "Lab 13: Core Enterprise Branch Network Architecture with SVI, EtherChannel & HSRP Redundancy"
description: "Step-by-step guide to building a redundant two-tier enterprise network in EVE-NG using LACP EtherChannel, SVI Inter-VLAN routing, Rapid-PVST+, and HSRP default gateway failover."
date: '2026-08-11T14:01:16+10:00'
tags: ["Cisco", "CCNA", "Networking", "EVE-NG", "Switching", "Routing", "HSRP", "EtherChannel"]
layout: "page"
---
<style>
  .max-w-prose, article, .prose { 
    max-width: 100% !important; 
    width: 100% !important; 
  }
</style>
---
## Executive Summary
This lab demonstrates how to design, build, and verify a highly available Core Enterprise Branch Network using Cisco routers and L2/L3 switches inside EVE-NG. Modern enterprise networks require continuous uptime, optimized bandwidth, and immediate failover capabilities. By implementing LACP EtherChannel, Switch Virtual Interfaces (SVIs) for Inter-VLAN routing, HSRP (Hot Standby Router Protocol) for default gateway redundancy, and Rapid-PVST+ for loop prevention, this lab creates a resilient two-tier network architecture capable of handling hardware and link failures without service disruption.

**Lab Objectives**
By completing this lab, you will be able to:

* Design a redundant two-tier enterprise network architecture inside EVE-NG.

* Configure 802.1Q VLAN Trunks and Access ports across Access and L3 Distribution switches.

* Bundle multiple physical interfaces into high-speed logical channels using LACP EtherChannel.

* Configure Switch Virtual Interfaces (SVIs) on L3 Distribution Switches to enable Inter-VLAN Routing.

* Deploy HSRP (Hot Standby Router Protocol) to provide virtual default gateway redundancy.

* Align Rapid-PVST+ Root Bridge placement with HSRP Active roles to optimize network traffic paths.

* Test and verify dynamic failover during primary switch and link outages.

## Network Topology & Addressing Table
**Topology Diagram**
```bash
[ Edge-Router ]
                      (10.0.0.1/30)
                            │
               ┌────────────┴────────────┐
         (10.0.0.2/30)             (10.0.0.6/30)
              Gi0/0                     Gi0/0
        [ Core-DSW1 ] ═══ LACP ═══ [ Core-DSW2 ]
         (HSRP Priority 110)        (HSRP Priority 100)
              Gi0/1                     Gi0/1
                │    \               /    │
             Trunk    \             /   Trunk
                │      \           /      │
              Gi0/1     Gi0/2   Gi0/1   Gi0/2
        [ Access-SW1 ] ═══ LACP ═══ [ Access-SW2 ]
          /        \                 /        \
       Fa0/10    Fa0/20           Fa0/10    Fa0/20
      (VLAN 10) (VLAN 20)        (VLAN 10) (VLAN 20)
         │          │               │          │
      [PC-10A]   [PC-20A]        [PC-10B]   [PC-20B]
```
**Topology Visual Design**
![Topology Design](/images/lab13/CoreEnterprise.png)

**IP Addressing Table**
```bash
| Device         | Interface    | IP Address / Subnet | Assigned Role / Description |
-------------------------------------------------------------------------------------
| *Core-DSW1**  | SVI VLAN 10  | `192.168.10.2/24`   | Active Gateway for VLAN 10  |

| **Core-DSW1**  | SVI VLAN 20  | `192.168.20.2/24`   | Standby Gateway for VLAN 20 |

| **Core-DSW2**  | SVI VLAN 10  | `192.168.10.3/24`   | Standby Gateway for VLAN 10 |

| **Core-DSW2**  | SVI VLAN 20  | `192.168.20.3/24`   | Active Gateway for VLAN 20  |

| **Virtual GW** | VLAN 10 HSRP | `192.168.10.1/24`   | Virtual Gateway for Sales (PC-10A/PC-10B) |

| **Virtual GW** | VLAN 20 HSRP | `192.168.20.1/24`   | Virtual Gateway for HR (PC-20A/PC-20B) |

| **PC-10A**     | Eth0         | `192.168.10.10/24`  | Host in VLAN 10 (Sales Department) |

| **PC-20A**     | Eth0         | `192.168.20.10/24`  | Host in VLAN 20 (HR Department) |
---