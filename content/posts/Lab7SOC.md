---
title: "Lab 7: Building an Enterprise SOC Architecture with Palo Alto Firewall & Wazuh SIEM"
description: "Deploying a multi-zone Palo Alto network topology, configuring Wazuh SIEM monitoring, and executing SSH Brute-Force simulation with Kali Linux."
date: 2026-07-18
tags: ["Palo Alto", "Wazuh", "SIEM", "SOC", "Hydra", "Network Security"]
layout: "page"
---
<style>
  .max-w-prose, article, .prose { 
    max-width: 100% !important; 
    width: 100% !important; 
  }
</style>

## 🎯 Executive Summary
The purpose of this lab is to design, deploy, and test a mini-enterprise security infrastructure. This project simulates real-world security engineering tasks by connecting a network security control platform with a centralized detection mechanism. 

### Key Milestones Achieved:
* **Network Segmentation:** Built three distinct security zones (Inside, DMZ, Outside) using a Palo Alto Next-Generation Firewall to control traffic flow.
* **Centralized Monitoring:** Deployed a Wazuh SIEM Server inside the DMZ zone and successfully configured log forwarding from internal endpoints.
* **Threat Simulation:** Executed an active SSH Brute-Force attack using Kali Linux and verified that security alerts were properly detected and visible on the Wazuh Dashboard in real-time.
---
![Lab Topology](/images/lab7/image1.png)

## 🛠️ Phase 1: Palo Alto NGFW Core Network Configuration

### 1.1 Security Zone Architecture
To maintain zero-trust segmentation, three distinct Layer 3 Security Zones were configured inside the Palo Alto Firewall:
*   **Inside-Zone:** Trusted corporate network pool.
*   **DMZ-Zone:** Semi-trusted segment isolating the monitoring infrastructure.
*   **Outside-Zone:** Untrusted public/internet egress point.

### 1.2 Physical Interface Mapping
![Lab Topology](/images/lab7/image2.png)
The topology's interfaces were mapped to their respective security layers and network configuration parameters:

1.  **Ethernet1/1 (Public Egress):**
    *   **Type:** Layer 3 | **Virtual Router:** Default
    *   **Security Zone:** Outside-Zone
    *   **IPv4 Allocation:** DHCP Client (Dynamic WAN Gateway)
    ![Lab Topology](/images/lab7/image3.png)

    ---
    ![Lab Topology](/images/lab7/image4.png)
2.  **Ethernet1/2 (Wazuh SIEM Link):**
    *   **Type:** Layer 3 | **Virtual Router:** Default
    *   **Security Zone:** DMZ-Zone
    *   **IPv4 Allocation:** `192.168.10.1/24` (Static Gateway for DMZ)
![Lab Topology](/images/lab7/image5.png)

3.  **Ethernet1/3 (Corporate Segment):**
    *   **Type:** Layer 3 | **Virtual Router:** Default
    *   **Security Zone:** Inside-Zone
    *   **IPv4 Allocation:** `10.1.1.1/24` (Static Gateway for LAN)
![Lab Topology](/images/lab7/image6.png)

### 1.3 Routing & NAT Matrix
To achieve routing visibility and facilitate secure internet transit:
![Lab Topology](/images/lab7/image7.png)
*   **Default Route Configuration:** A static route (`0.0.0.0/0`) was anchored to `ethernet1/1` targeting the upstream next-hop gateway at `192.168.64.2`.
![Lab Topology](/images/lab7/image8.png)
*   **Source NAT (PAT Execution):** Formulated an `Internet-NAT` policy mapping source packets from `Inside-Zone` and `DMZ-Zone` onto `ethernet1/1` using **Dynamic IP and Port (Interface Address)** translation.
![Lab Topology](/images/lab7/image9.png)
![Lab Topology](/images/lab7/image10.png)

### 1.4 Firewall Security Rule Base
The firewall security architecture defines rule bases to allow required communication flows:

| Rule Name | Source Zone | Destination Zone | Action | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Allow-Outbound** | Inside-Zone, DMZ-Zone | Outside-Zone | Allow | Permits Internet access for internal devices. |
| **Agent-to-Wazuh** | Inside-Zone | DMZ-Zone | Allow | Allows log forwarding from endpoints to SIEM. |
| **Admin-to-Wazuh-GUI**| Inside-Zone | DMZ-Zone | Allow | Permits administrative HTTPS control plane access. |

![Lab Topology](/images/lab7/image11.png)

---

## 🐧 Phase 2: Deploying Wazuh SIEM Server & Endpoint Provisioning

### 2.1 Ubuntu Server Networking via Netplan
The Wazuh instance was provisioned with dual NIC topologies: `ens3` (Direct Internet access for packaging updates) and `ens4` (Connected to Palo Alto DMZ segment). 

Network parameters were declared inside `/etc/netplan/00-installer-config.yaml`:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      dhcp4: true
    ens4:
      dhcp4: false
      addresses:
        - 192.168.10.10/24
```
### 2.2 Core SIEM Automated Installation
Executed the native deployment script to initilize the indexer, dashboard, and management components.
```bash
curl -sO [https://packages.wazuh.com/4.14/wazuh-install.sh](https://packages.wazuh.com/4.14/wazuh-install.sh) && sudo bash wazuh-install.sh -a
```
![Lab Topology](/images/lab7/image12.png)

Once the installation is complete, make sure to copy the automatically generated username and temporary long password for your initial login.

![Lab Topology](/images/lab7/image13.png)
![Lab Topology](/images/lab7/image14.png)

#### Wazuh Manager is working now
![Lab Topology](/images/lab7/image15.png)

### 2.3 Endpoint Agent Installation (Ubuntu Client)
To enroll the internal `Ubuntu-Desktop-Client` into the monitoring node, the management address arguement was specified during binary registration.

#### First we need to assign ip address on Ubuntu Desktop
![Lab Topology](/images/lab7/image16.png)

#### Check ip address and ping to 8.8.8.8
```bash
ip a | grep ens3
```
![Lab Topology](/images/lab7/image17.png)

An initial ping request to the Wazuh Server at `192.168.10.10` was unsuccessful. This issue occurred because the dynamic/static route had not been added to the Ubuntu Server's routing table.
![Lab Topology](/images/lab7/image18.png)

#### Finally Ping Works
![Lab Topology](/images/lab7/image19.png)

```bash
wget [https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.6-1_amd64.deb](https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.6-1_amd64.deb) && \
sudo WAZUH_MANAGER='192.168.10.10' WAZUH_AGENT_NAME='Ubuntu-Desktop-Client' dpkg -i ./wazuh-agent_4.14.6-1_amd64.deb
```
![Lab Topology](/images/lab7/image20.png)

![Lab Topology](/images/lab7/image21.png)
![Lab Topology](/images/lab7/image22.png)
![Lab Topology](/images/lab7/image23.png)

#### Checking Wazuh-Agent Status
```bash
sudo systemctl status wazuh-agent
```
![Lab Topology](/images/lab7/image24.png)
#### The Wazuh Agent is now actively enrolled and connected to the Wazuh Manager.
![Lab Topology](/images/lab7/image25.png)
---
![Lab Topology](/images/lab7/image26.png)
---
![Lab Topology](/images/lab7/image27.png)

### Setting Up Kali Attacking Machine
Configuring Kali Linux as the Attack PC requires setting up its network interface. Run the following commands to assign a static IP address to eth0, bring the link state up, and append the default route targeting the internal network segment.
![Lab Topology](/images/lab7/image28.png)

```bash
sudo ip addr add 10.1.1.50/24 dev eth0
sudo ip link set eth0 up
sudo ip route add default via 10.1.1.1 dev eth0
```
#### Checing IP Address
```bash
ip a | grep eth0
```
![Lab Topology](/images/lab7/image29.png)

#### Ping to Wazuh-Agent
```bash
ping 10.1.1.10 -c 4
```
![Lab Topology](/images/lab7/image30.png)
Sometimes, updating Kali Linux fails despite having a proper network route in place. The root cause is a DNS resolution failure, meaning domain names cannot be translated into IP address.

![Lab Topology](/images/lab7/image31.png)
To resolve this above issue, I need to update your DNS configuration in `/etc/resolv.conf`. Execute 
`sudo nano /etc/resolv.conf`and 
add `nameserver 8.8.8.8` and `nameserver 1.1.1.1`. Save the changes, exit the editor, and test the resolution by running a ping request to `google.com`.

![Lab Topology](/images/lab7/image32.png)
![Lab Topology](/images/lab7/image33.png)

### SOC Attack Simulation
The main goal of this lab is to understand how a real-world Security Operations Center (SOC) works. You will learn how a SOC Analyst tracks cyberattacks and how security alerts appear on the SIEM (Wazuh) dashboard.

#### Attack 1: 
#### Attack Method: `SSH Brute Force Attack`

A Brute Force attack is one of the simplest types of cyberattacks. The attacker tries to guess the correct password of a target machine by automatically testing many words from a password list. We choose SSH (Port 22) because many Linux servers and desktops leave this port open for remote access.

#### Attack Tool: `Hydra`
Hydra is a very fast and powerful tool used for brute-forcing network logins. It can send multiple requests at the same time (parallel connections) to test hundreds of passwords in just a few seconds. In this attack, Hydra uses a famous password file called rockyou.txt, which contains over 14 million common passwords, to guess the login credentials one by one.
![Lab Topology](/images/lab7/image34.png)
![Lab Topology](/images/lab7/image35.png)

Sometimes, the `rockyou.txt` file is compressed as a zip file named `rockyou.txt.gz`. You need to use the gunzip command to extract it.

#### Command Explanation

* `-l user`: Specifies the target username as "user" to guess the password.  
* `-P `/usr/share/wordlists/rockyou.txt: Points to the path of the password list used for the attack.  
* `ssh://10.1.1.10`: Sets the target IP address and the protocol (SSH) for the attack.  
* `-t 4`: Sets the speed of the attack by running 4 tasks at the same time.  
* `-V`: Shows each password attempt on the terminal screen in real-time (Verbose mode).  

In the previous step, we guessed the username as `"user"`. Now, we will try again by changing the username to `"root"`

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://10.1.1.10 -t 4 -V
```
![Lab Topology](/images/lab7/image36.png)

### Now we can see the incoming alerts on Wazuh Dashboard.
![Lab Topology](/images/lab7/image37.png)

The Attack PC is still running the brute force attack, and logs are continuously appearing in Wazuh.

![Lab Topology](/images/lab7/image38.png)

### Checking auth.log on Victim Machine
```bash
sudo tail -f /var/log/auth.log
```
![Lab Topology](/images/lab7/image39.png)

---

## 🏁 Conclusion & Key Takeaways
This lab successfully models how a Security Operations Center (SOC) detects network-level and host-level attacks. By integrating a perimeter firewall with a host-based SIEM platform, we achieved complete visibility over the environment.

### Operational Insights:
1. **Defense-in-Depth:** Segmenting the Wazuh Server into a dedicated DMZ ensures that even if an internal client is compromised, the management infrastructure remains isolated and protected.
2. **Telemetry Validation:** Host-level log inspection (`auth.log`) confirmed that high-frequency authentication failures directly map to automated brute-force scripts. Wazuh successfully converted these raw system logs into high-priority actionable alerts.
3. **Infrastructure Readiness:** Encountering and fixing network route issues and DNS failures during the setup emphasized the importance of baseline network troubleshooting before deploying security tooling.