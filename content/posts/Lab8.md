---
title: "Lab 8: Enterpise Group Policy Management & Troubleshooting | Part: 1"
description: "Deployment and administration of a small enterprise network infrastructure using Microsoft Active Directory, Cisco networking devices, and Palo Alto Next-Generation Firewall"
date: '2026-07-21T16:25:50+10:00'
tags: ["Palo Alto", "Routing", "Switching", "WSUS", "AD DS", "Microsoft Server"]
layout: "page"
---
<style>
  .max-w-prose, article, .prose { 
    max-width: 100% !important; 
    width: 100% !important; 
  }
</style>

## Part 1: Network Configuration

## Executive Summary

This lab demonstrates the deployment and administration of a small enterprise network infrastructure using Microsoft Active Directory, Cisco networking devices, and Palo Alto Next-Generation Firewall.

This environment was designe to simulate a real-world corporate network consisting of multiple departmetns, segmented VLANs, centralized identity management, security policies, and endpoint management.

The objective as to deploy a secure Windows Infrastructure, implement Group Policy controls, enforce network segmentation, and troubleshoot common enterprise administration issues.

### Lab Objectives

- Deploy Active Directory Domain Servvices (AD DS)
- Configure DNS and DHCP Services
- Implement Department-Based VLAN Segmentation
- Configure Cisco Layer 2 Swittching
- Configure Palo Alto Firewall Security Policies
- Join Multiple Windows Endpoints to Active Directory
- Deploy and Validate Group Policy Objects (GPOs)

## Enterprise Topology

![Topology.png](/images/Lab8/Topology1.png)


### IP Address Plan


| Device | Role | IP Address |
| --- | --- | --- |
| R1 | Internet Router | 192.168.64.2 |
| Palo Alto Outside | WAN Interface | DHCP |
| Palo Alto VLAN10 | Server Gateway | 10.1.10.1 |
| Palo Alto VLAN20 | HR Gateway | 10.1.20.1 |
| Palo Alto VLAN30 | IT Gateway | 10.1.30.1 |
| Palo Alto DMZ | DMZ Gateway | 192.168.40.1 |
| DC01 | Domain Controller | 10.1.10.10 |
| HR-PC01 | HR Workstation | 10.1.20.10 |
| HR-PC02 | HR Workstation | 10.1.20.11 |
| IT-PC01 | IT Workstation | 10.1.30.10 |
| IT-PC02 | IT Workstation | 10.1.30.11 |
| WSUS01 | Patch Management Server | 192.168.40.10 |
| FILE01 | File Server | 192.168.40.20 |

### Router Configuration

```cmd

Router>en
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname myRouter
myRouter(config)#int e0/0
myRouter(config-if)#ip address dhcp
---
myRouter#show ip int brief 
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                192.168.64.148  YES DHCP   up                    up      
Ethernet0/1                unassigned      YES unset  administratively down down    
Ethernet0/2                unassigned      YES unset  administratively down down    
Ethernet0/3                unassigned      YES unset  administratively down down  

```

### Ethernet 0/1  connect to Palo Alto Firewall

```cmd
myRouter>
myRouter>en
myRouter#conf t
myRouter(config)#int e0/1
myRouter(config-if)#ip address 192.168.100.100 255.255.255.0
myRouter(config-if)#no shut
myRouter(config-if)#exit
myRouter(config)#
myRouter#sh ip int brief 
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                192.168.64.148  YES DHCP   up                    up      
Ethernet0/1                192.168.100.100 YES manual up                    up      
Ethernet0/2                unassigned      YES NVRAM  administratively down down    
Ethernet0/3                unassigned      YES NVRAM  administratively down down    
myRouter#

```

### Static Routing & NAT on Cisco Router

```cmd
# Assign Static Routes to Palo Alto Outside Interface IP
myRouter(config)#ip route 10.1.10.0 255.255.255.0 192.168.100.1
myRouter(config)#ip route 10.1.20.0 255.255.255.0 192.168.100.1  
myRouter(config)#ip route 10.1.30.0 255.255.255.0 192.168.100.1 
myRouter(config)#ip route 192.168.40.0 255.255.255.0 192.168.100.1 

# Internet Overload NAT on Router
myRouter(config)#access-list 1 permit 192.168.100.0 0.0.0.255
myRouter(config)#access-list 1 permit 10.1.0.0 0.0.255.255
myRouter(config)#access-list 1 permit 192.168.40.0 0.0.0.255

myRouter(config)#int e0/0
myRouter(config-if)#ip nat outside
myRouter(config-if)#exit
myRouter(config)#
*Jul 20 16:33:21.633: %LINEPROTO-5-UPDOWN: Line protocol on Interface NVI0, changed state to up

myRouter(config)#int e0/1
myRouter(config-if)#ip nat inside
myRouter(config-if)#exit

myRouter(config)#ip nat inside source list 1 interface e0/0 overload
myRouter(config)#end
myRouter#write
*Jul 20 16:34:51.117: %SYS-5-CONFIG_I: Configured from console by console
myRouter#write memory
Building configuration...
[OK]
myRouter#

```

### Palo Alto Firewall Configuration

Screenshot of Initial Configuration

![image.png](/images/Lab8/image.png)

![image.png](/images/Lab8/image1.png)

1. **Outside Interface (ethernet1/1) Configuration**

Purpose: Connecting from Palo Alto to Cisco Router throught `e0/1` `(192.168.100.100)` to get internet access.

- Palo Alto Web UI - > `Network Tab` → `Interface` → `Ethernet`
- Click on `ethernet1/1`

![image.png](/images/Lab8/image2.png)

- Under Config Tab:
    - Virtual Router: `default`
    - Security Zone: click on `New Zone`  Name Tab `Outside-Zone` Type `Layer3`
    - Click on `OK`

![image.png](/images/Lab8/image3.png)

- IPv4 Tab
    - Click on `Add` type `192.168.100.1/24`
- `OK`

![image.png](/images/Lab8/image4.png)

1. **Static Default Route Configuration**

**Purpose**: To route all unknown traffic from the Palo Alto firewall to the Cisco Router.

- Network Tab - > `Virtual Routers`

![image.png](/images/Lab8/image5.png)

- Click on `default`
- Go to `Static Routes` Tab and click on `Add`
    - Name: `Default-Internet-Route`
    - Destination: `0.0.0.0/0`
    - Interface: `ethernet1/1`
    - Next Hop: `IP Address` Put IP address of Cisco Router `192.168.100.100`
- OK - > `OK`

![image.png](/images/Lab8/image6.png)

### Configuring DMZ Interface (ethernet1/2)

**Purpose:** To configure Cisco `SW1` to act as the Gateway for `WSUS01` and `FILE01` (DMZ Subnet 192.168.40.0/24)

1. Network Tab - > `Interfaces` - > `Ethernet` → click on `ethernet1/2`
2. Config Tab:
    - Interface Type: `Layer3`
    - Virtual Router: `default`
    - Security Zone: Choose `New Zone`

![image.png](/images/Lab8/image7.png)

- Name : `DMZ-Zone`  → Click on `OK`

![image.png](/images/Lab8/image8.png)

1. IPv4 Tab:
- Click on `Add` → type `192.168.40.1/24`
- Click on `OK`

![image.png](/images/Lab8/image9.png)

### Configuring Inside Trunk Interfaces & Sub-Interfaces `ethernet1/3`

**Purpose:** To separate incoming traffic from VLAN 10, 20, and 30 on Cisco SW2's e1/2 Trunk Port according to their VLAN tags, and assign Gateway IP addresses to each (Router-on-a-Stick Concept)

1. Network Tab → `Interfaces` → `ethernet1/3`
- Interface Type → `Layer3`
- Config Tab → `New Zone`
- Type `Inside-Zone`
- Click on `OK`

![image.png](/images/Lab8/image10.png)

![image.png](/images/Lab8/image11.png)

IPv4 Tab → `Add`

Assign IP Address `10.1.10.1/24`

Click on `OK`

![image.png](/images/Lab8/image12.png)

![image.png](/images/Lab8/image13.png)

### Making 3 Sub-Interfaces

Select `ethernet1/3` and click on `Add Subinterface` 

![image.png](/images/Lab8/image14.png)

![image.png](/images/Lab8/image15.png)

Creating Sub-Interface 2

![image.png](/images/Lab8/image16.png)

![image.png](/images/Lab8/image17.png)

Creating Sub-Interface 3

![image.png](/images/Lab8/image18.png)

![image.png](/images/Lab8/image19.png)

To apply changes click on `Commit` 

![image.png](/images/Lab8/image20.png)

![image.png](/images/Lab8/image21.png)
---
### Cisco SW1 Configuration

```cmd
# Assign Hostname
Switch>en
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname SW1

```

```cmd
# Creating VLAN Databases
SW1(config)#vlan 10
SW1(config-vlan)#name SERVER
SW1(config-vlan)#vlan 20
SW1(config-vlan)#name HR
SW1(config-vlan)#vlan 30
SW1(config-vlan)#name IT
SW1(config-vlan)#vlan 40
SW1(config-vlan)#name DMZ
SW1(config-vlan)#exit
SW1(config)#

```

#### Configuring `e0/0` interface connecting to Palo Alto Firewall

This opens a trunk connection between the two switches to allow `VLAN 10,20, and 30` traffic from `SW2` to reach Palo Alto firewall.

```cmd
SW1(config)#int e0/0
SW1(config-if)#description Link-to-PaloAlto-eth1/2
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 40
SW1(config-if)#no shut
SW1(config-if)#exit
SW1(config)#

```

#### Configuring the Inter-Switch Trunk Link `e0/1` to `SW2`

```cmd

SW1(config)#int e0/1
SW1(config-if)#description Inter-Switch-Trunk-to-SW2-e0/0
SW1(config-if)#switchport trunk encapsulation dot1q
SW1(config-if)#switchport mode trunk
SW1(config-if)#no shut
SW1(config-if)#exit
SW1(config)#

```

#### Configuring Access Ports for DMZ servers (WSUS01 & FILE01).

Ports `e0/2 (WSUS01)` and `e0/3 (File01)` will be assigned to `VLAN 40`

```cmd
SW1(config)#int range e0/2 - 3
SW1(config-if-range)#description DMZ-Servers-WSUS01-FILE01
SW1(config-if-range)#switchport mode access
SW1(config-if-range)#switchport access vlan 40
SW1(config-if-range)#no shut
SW1(config-if-range)#exit
SW1(config)#

```

#### Saving the `Cofiguration`

```cmd
SW1(config)#end
SW1#write memory
Building configuration...
Compressed configuration from 1106 bytes to 723 bytes[OK]
SW1#

```

#### Checking Trunk Port working or not!

```cmd
SW1#show int trunk 

Port        Mode             Encapsulation  Status        Native vlan
Et0/1       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Et0/1       1-4094

Port        Vlans allowed and active in management domain
Et0/1       1,10,20,30,40

Port        Vlans in spanning tree forwarding state and not pruned
Et0/1       1,10,20,30,40

```

![image.png](/images/Lab8/image22.png)

---

#### Cisco SW2 Step by Step Configuration

| **Port** | **Destination / Device** | **Mode** | **Network / Description** |
| --- | --- | --- | --- |
| **`e1/2`** | Palo Alto `eth1/3` | **Trunk Link** | Inside VLANs (10, 20, 30) to Gateway |
| **`e0/0`** | Cisco SW1 `e0/1` | **Trunk Link** | Inter-Switch Link |
| **`e0/1`** | DC01 | **Access Link** | VLAN 10 (SERVER) |
| **`e0/2`** | HR-PC01 | **Access Link** | VLAN 20 (HR) |
| **`e0/3`** | HR-PC02 | **Access Link** | VLAN 20 (HR) |
| **`e1/0`** | IT-PC01 | **Access Link** | VLAN 30 (IT) |
| **`e1/1`** | IT-PC02 | **Access Link** | VLAN 30 (IT) |
|  |  |  |  |

```cmd
# Assign hostname
Switch>en
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname SW2
SW2(config)#

```

To enable the transfer of both the `inside` VLANS (intended for use on `SW2`) and DMZ traffic between the switches, I will configure VLANs 10, 20, 30, and 40.

```cmd
SW2(config)#vlan 10     
SW2(config-vlan)#name SERVER
SW2(config-vlan)#vlan 20
SW2(config-vlan)#name HR
SW2(config-vlan)#vlan 30
SW2(config-vlan)#name IT
SW2(config-vlan)#vlan 40
SW2(config-vlan)#name DMZ
SW2(config-vlan)#exit
SW2(config)#
```

Trunk Ports Configuration ( Palo Alto and SW1)

```cmd
SW2(config)#int range e0/0, e1/2
SW2(config-if-range)#description Trunk-Links-to-SW1-and-PaloAlto
SW2(config-if-range)#switchport trunk encapsulation dot1q
SW2(config-if-range)#switchpor
*Jul 21 16:04:33.581: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet1/2, changed state to down
SW2(config-if-range)#switchport mode trunk
*Jul 21 16:04:36.581: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet1/2, changed state to up
SW2(config-if-range)#switchport mode trunk
SW2(config-if-range)#no s
*Jul 21 16:04:39.860: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet1/2, changed state to down
SW2(config-if-range)#no shut
SW2(config-if-range)#
*Jul 21 16:04:42.867: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet1/2, changed state to up
SW2(config-if-range)#exit
SW2(config)#

```
* Screenshot
![image.png](/images/Lab8/image23.png)

Assigning `VLAN 10` to the Active Directory Server (`DC01` port

```cmd
SW2(config)#int e0/1
SW2(config-if)#description Access-Port-DC01-Server
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 10
SW2(config-if)#no shut
SW2(config-if)#exit
SW2(config)#

```

Assigning `VLAN 20` to `HR Department`  Workstations Ports

```cmd
SW2(config)#int range e0/2 - 3
SW2(config-if-range)#description Access-Ports-HR-Workstations
SW2(config-if-range)#switchport mode access
SW2(config-if-range)#switchport access vlan 20
SW2(config-if-range)#no shut
SW2(config-if-range)#exit
SW2(config)#

```

Assigning `VLAN 30` to `IT Department` Workstations Ports

```cmd
SW2(config)#int range e1/0 - 1
SW2(config-if-range)#description Access-Ports-IT-Workstations
SW2(config-if-range)#switchport mode access
SW2(config-if-range)#switchport access vlan 30
SW2(config-if-range)#no shut
SW2(config-if-range)#exit
SW2(config)#

```

Saving Configuration

```cmd
SW2(config)#end
SW2#w
*Jul 21 16:18:10.861: %SYS-5-CONFIG_I: Configured from console by console
SW2#write memory
Building configuration...
Compressed configuration from 1553 bytes to 886 bytes[OK]

```

Verification

```cmd
SW2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et1/3
10   SERVER                           active    Et0/1
20   HR                               active    Et0/2, Et0/3
30   IT                               active    Et1/0, Et1/1
40   DMZ                              active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
SW2#

```

```cmd
SW2#show int trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      1
Et1/2       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Et0/0       1-4094
Et1/2       1-4094

Port        Vlans allowed and active in management domain
Et0/0       1,10,20,30,40
Et1/2       1,10,20,30,40

Port        Vlans in spanning tree forwarding state and not pruned
Et0/0       1,10,20,30,40
Et1/2       1,10,20,30,40
SW2#

```