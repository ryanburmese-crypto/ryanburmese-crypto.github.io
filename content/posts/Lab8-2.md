+++
title = "Lab 8: Enterprise Group Policy Management & Troubleshooting | Part 2"
date = "2026-07-23"
author = "Ryan Burman"
tags = ["Active Directory", "Palo Alto", "Windows Server", "DHCP Relay", "Group Policy", "Networking"]
categories = ["Tech Lab", "Cybersecurity"]
draft = false
layout= "page"
+++

<style>
  .max-w-prose, article, .prose { 
    max-width: 100% !important; 
    width: 100% !important; 
  }
</style>

## Executive Summary

Following the network backbone deployment in Part 1, this lab focuses on building core identity and management services for an enterprise network. I deployed Microsoft Active Directory Domain Services (AD DS), DNS, and centralized DHCP management on Windows Server (`DC01`). 

To enable dynamic IP assignment across segmented network boundaries, I configured DHCP Relay on the Palo Alto Next-Generation Firewall to bridge cross-VLAN requests. Furthermore, Inter-VLAN security policies and Source NAT were established to ensure secure traffic flow between internal zones, DMZ servers, and the Internet. Finally, workstation endpoints were successfully integrated into the Active Directory domain (`corp.local`) and validated via domain user authentication.

---

## Phase 1: Active Directory Domain Controller (`DC01`) Setup

### Step 1: Static IP & DNS Assignment

Assign the following static IP configuration to `DC01`:

* **IP Address:** `10.1.10.10`
* **Subnet Mask:** `255.255.255.0`
* **Default Gateway:** `10.1.10.1` (Palo Alto `eth1/3.10` IP)
* **Preferred DNS Server:** `127.0.0.1` or `10.1.10.10`

![Static IP Assignment](/images/lab8-2/image.png)

#### Troubleshooting ICMP/Ping Requests to Firewall
An initial ICMP ping request from `DC01` to the Palo Alto default gateway (`10.1.10.1`) failed because Palo Alto blocks incoming ICMP management traffic by default.

![Failed Ping Request](/images/lab8-2/image1.png)

**Resolution Steps:**
1. Navigate to Palo Alto Web UI → **Network** tab → **Network Profiles** → **Interface Mgmt**.
2. Click **Add**:
   * **Name:** `Allow-Ping`
   * **Administrative Management Services:** Check `Ping`
3. Click **OK**.

![Interface Management Profile](/images/lab8-2/image2.png)

4. Go to **Network** → **Interfaces** → **Ethernet**.
5. Double-click `ethernet1/3.10` (VLAN 10 Gateway).
6. Navigate to the **Advanced** tab, click **Management Profile**, and select `Allow-Ping`.
7. Click **OK** and execute a **Commit** to apply the policy.

![Assign Management Profile](/images/lab8-2/image3.png)

*Verification:* ICMP echo requests to the firewall gateway now succeed.

![Successful Ping Test](/images/lab8-2/image4.png)

---

### Step 2: AD DS, DNS & DHCP Role Installation

1. Open **Server Manager** → **Manage** → **Add Roles and Features**.
2. Under **Server Roles**, select the following roles:
   * Active Directory Domain Services
   * DNS Server
   * DHCP Server
3. Click **Install** and wait for completion.

![Add Roles and Features](/images/lab8-2/image5.png)
![Select Server Roles](/images/lab8-2/image6.png)
![Role Feature Selection](/images/lab8-2/image7.png)
![Confirm Installation Options](/images/lab8-2/image8.png)
![Role Installation Progress](/images/lab8-2/image9.png)

---

### Step 3: Domain Controller Promotion (`corp.local`)

1. Click the **Notification Flag ⚠️** in Server Manager and select **Promote this server to a domain controller**.
2. **Deployment Configuration:** Select **Add a new forest** and enter Root domain name: `corp.local`.
3. Enter a secure **DSRM Password**, click **Next**, and follow the wizard defaults.
4. Click **Install**. The server will reboot automatically upon completion.

![Add New Forest](/images/lab8-2/image10.png)
![Domain Options and DSRM Password](/images/lab8-2/image11.png)
![Promote DC Installation](/images/lab8-2/image12.png)
![Automatic System Reboot](/images/lab8-2/image13.png)

---

## Phase 2: Organizational Units (OUs) & User Accounts Setup

Target Active Directory Organizational Structure:

```text
corp.local
├── 📂 Corp Users
│   ├── 📂 HR Department
│   └── 📂 IT Department
├── 📂 Workstations
│   ├── 📂 HR PCs
│   └── 📂 IT PCs
└── 📂 Servers
```
Once the domain controller reboots, log in using `CORP\Administrator` credentials and launch Active Directory Users and Computers (`dsa.msc`).

![Booting Up As Corp.local](/images/lab8-2/image14.png)

**Step 1:** Creating OU Hierarchy
Right-click `corp.local` → `New` → `Organizational Unit` and create the OU tree as specified above.

![Booting Up As Corp.local](/images/lab8-2/image15.png)

**Step 2: Create Department User Accounts**

* HR Department OU: Create users hruser01 and hruser02.  
* IT Department OU: Create users ituser01 and ituser02.

**Note:** Set administrative passwords and check "Password never expires" for lab testing.  

![Crating OUs and users](/images/lab8-2/image16.png)

**Phase 3: DHCP Scopes Setup (`VLAN 20 HR` & `VLAN 30 IT`)**

Open DHCP Manager (`dhcpmgmt.msc`) on `DC01` to configure IP distribution for client endpoints:

**Scope 1: VLAN 20 (HR Subnet)**

- Scope Name: `VLAN20-HR`
- Start IP: `10.1.20.100` | End IP: `10.1.20.150`
- Subnet Mask: `255.255.255.0` (`/24`)
- Router (Default Gateway): `10.1.20.1`
- DNS Server: `10.1.10.10`

![DHCP Scope for VLAN20](/images/lab8-2/image17.png)
![DHCP Scope for VLAN20](/images/lab8-2/image18.png)

**Scope 2: VLAN 30 (IT Subnet)**

- Scope Name: `VLAN30-IT`
- Start IP: `10.1.30.100` | End IP: `10.1.30.150`
- Subnet Mask: `255.255.255.0` (`/24`)
- Router (Default Gateway): `10.1.30.1`
- DNS Server: `10.1.10.10`

![DHCP Scope for VLAN30](/images/lab8-2/image19.png)

**Authorize DHCP Server:** 
Right-click the server instance in DHCP Manager and select Authorize to allow IP leases within Active Directory.

![Authorize](/images/lab8-2/image20.png)
---

**Set up DHCP Relay on Palo Alto**

When Client PCs in VLAN 20 (HR) and VLAN 30 (IT) ask for a dynamic IP, the Palo Alto Gateway needs to forward (relay) those requests to DC01 (DHCP Server: 10.1.10.0) in the VLAN 10 Subnet.

- Log in to the Palo Alto Web UI → Go to `Network` Tab → Click `DHCP`.
- Click the `DHCP Relay`tab at the top, then click `Add` at the bottom:
    - Interfaces: Add `ethernet1/3.20 (HR)` and `ethernet1/3.30 (IT)`
    - DHCP Server IP: Under DHCP Server Addresses on the right, type `10.1.10.10`.
- Click `OK`.
![DHCP Relay](/images/lab8-2/image21.png)

**Palo Alto Inter-VLAN & Internet Security Policies**

By default, Palot Alto blocks all traffic between different zones. We need to create Security Policies so that the Inside Zone, DMZ Zone, and Outside Zone can communicate with each other.

Go to the `Policies`tab → `Security`, and create the following 3 rules;

**Rule 1:** Allow-Internal-InterVLAN (To allow communication between Inside VLANs)

- Name: `Allow-Internal-InterVLAN`
- Source Zone: `Inside-Zone`
- Destination Zone: `Inside-Zone`
- Application / Service: `any` / `application-default`
- Action: `Allow`
![Policies](/images/lab8-2/image22.png)

**Rule 2:** Allow-Inside-to-DMZ (To allow Inside Zone to reach DMZ Servers)

- Name: `Allow-Inside-to-DMZ`
- Source Zone: `Inside-Zone`
- Destination Zone: `DMZ-Zone`
- Application / Service: `any` / `application-default`
- Action: `Allow`

**Rule 3:** Allow-Inside-to-Internet (To allow internal users to access the internet)

- Name: `Allow-Inside-to-Internet`
- Source Zone: `Inside-Zone`
- Destination Zone: `Outside-Zone`
- Application / Service: `any` / `application-default`
- Action: `Allow`

![Policies](/images/lab8-2/image23.png)

**Set up Palo Alto Source NAT (For Internet Access)**

Go to the `Policies` tab → Click `NAT` → Click `Add`:

- General Tab:
    - Name: `Inside-to-Outside-NAT`
- Original Packet Tab:
    - Source Zone: `Inside-Zone`
    - Destination Zone: `Outside-Zone`
    - Destination Interface: `ethernet1/1`
- Translated Packet Tab:
    - Source Address Translation:
        - Translation Type: `Dynamic IP and Port`
        - Address Type: `Interface Address`
        - Interface: `ethernet1/1`

Click `OK` 

Click `Commit` in the top right corner to save and apply your changes.
![NAT Policies](/images/lab8-2/image24.png)

**Phase 5:** Endpoint Integration & Domain Join Verification
**Joining Enpoints to the Domain**

After completing the settings above, Now we can join the Client PCs to the domain.

#### Check IP and Connection on HR-PC01 / IT-PC01

- Log in to **HR-PC01** (or **IT-PC01**) and open the Command Prompt. Type `ipconfig /renew` and press Enter.
- Check if you received an IP address in the `10.1.20.x` range (or `10.1.30.x`) from the DHCP Server (DC01).
    - *Note: The DNS address must be `10.1.10.10`.*
- Test the connection using `ping`:
    - Type `ping corp.local` (it should successfully reach the domain name).

**Screenshot of HR-PC01**’s IP
![NAT Policies](/images/lab8-2/image25.png)

#### Join the Domain

- On the Client PC, press **Win + R**, type `sysdm.cpl`, and press **Enter**.
- Go to the **`Computer Name`** tab -> Click the **`Change`   ...** button.
- Under **Domain:** type `corp.local` and click **OK**.
- When prompted for credentials, enter an Active Directory Account or **`CORP\Administrator`** with its password.
- When you see the message `*Welcome to the corp.local domain*`, click **`Restart Now`**.
![Joining Domain](/images/lab8-2/image26.png)
![NAT Policies](/images/lab8-2/image27.png)

**Domain Logon Test**
After rebooting, sign into `HR-PC01` using domain credentials (`CORP\hruser01`)
![Log in](/images/lab8-2/image28.png)

**Conclusion**

In this second part of the lab, I successfully built the core identity and networking services for our enterprise network. By setting up Active Directory on DC01, I organized users and computers into structured Organizational Units (OUs) for better management.

I also solved a common multi-subnet challenge by configuring DHCP Relay on the Palo Alto firewall. This allowed computers in different VLANs (HR and IT) to automatically get IP addresses from a single central server.
Finally, by applying proper security policies and NAT rules on the firewall, client PCs were able to safely communicate across subnets, join the corp.local domain, and log in with department accounts.