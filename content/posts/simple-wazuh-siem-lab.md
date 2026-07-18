---
title: "Lab 4: Centralized Log Management with Wazuh SIEM"
date: 2026-07-14
description: "Setting up an enterprise-grade Wazuh SIEM/XDR architecture in EVE-NG with Cisco switching, multi-platform agents, and simulated SSH brute force detection."
tags: ["SIEM", "Wazuh", "Cybersecurity", "SOC", "Linux", "Windows Server", "EVE-NG"]
categories: ["Homelab Projects"]
showToc: true
TocOpen: false
keepImageRatio: true
---
<style>
  .max-w-prose, article, .prose { 
    max-width: 100% !important; 
    width: 100% !important; 
  }
</style>
## Executive Summary
In modern Security Operations Centers (SOCs), Centralized Log Management is the foundational pillar of visibility. This project demonstrates the deployment of **Wazuh SIEM/XDR**, an enterprise-ready open-source monitoring platform, within a virtualized lab environment (**EVE-NG**). 

The infrastructure features a managed Layer 2 network, a central SIEM Manager node, and dual-platform workloads (Windows Server and Ubuntu Desktop) configured to ship telemetry. The lab culminates in a successful detection and analysis of a simulated credential-stuffing (SSH Brute Force) attack.

---

## 🗺️ 1. Network Topology & IP Schema

The lab environment was orchestrated inside EVE-NG using a managed **Cisco vIOS-L2** switch for traffic isolation and upstream connectivity to the internet.

```text
               [ Internet / Management Cloud ] (NAT: 192.168.64.0/24)
                              │
                    [ Cisco vIOS-L2 Switch ]
               (e0/0 - Uplink to Cloud/Gateway)
                              │
         ┌────────────────────┼────────────────────┐
         │ e0/1               │ e0/2               │ e0/3
 ┌───────┴───────┐    ┌───────┴───────┐    ┌───────┴───────┐
 │ Wazuh Manager │    │ Windows Server│    │ Ubuntu Desktop│
 │   (Ubuntu)    │    │ (Active Agent)│    │ (Linux Agent) │
 ├───────────────┤    ├───────────────┤    ├───────────────┤
 │192.168.64.10  │    │192.168.64.20  │    │192.168.64.30  │
 └───────────────┘    └───────────────┘    └───────────────┘

```
---
 ### Network Topology Screenshot
![Network Topology](/images/NetworkTopology.png)
### 🌐 IP Allocation Table

| Hostname | Role | OS | IP Address | Subnet Mask / Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **Wazuh-Manager** | SIEM Core & Indexer | `Ubuntu Server 22.04 LTS` | `192.168.64.10/24` | `192.168.64.2` |
| **Windows-Target** | Monitored Client | `Windows Server 2022` | `192.168.64.20/24` | `192.168.64.2` |
| **Ubuntu-Desktop** | Monitored Client | `Ubuntu Desktop 22.04` | `192.168.64.30/24` | `(192.168.64.2)` |

## Static IP Assignment on Wazuh Manager (Netplan)
The Ubuntu-based SIEM server was assigned a static IP using Netplan configuration at 
/etc/netplan/00-installer-config.yaml
```bash
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.64.10/24]
      routes:
        - to: default
          via: 192.168.64.2
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```
### Apply config and enforce file permissions:
```bash
sudo chmod 600 /etc/netplan/00-installer-config.yaml
sudo netplan apply
```
### Static IP Assigned and Ping Test
```bash
root@ubuntu:~# ip a | grep ens3
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.64.10/24 brd 192.168.64.255 scope global ens3
    inet 192.168.64.141/24 metric 100 brd 192.168.64.255 scope global secondary dynamic ens3
root@ubuntu:~# ping 8.8.8.8 -c 4
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=128 time=9.78 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=128 time=12.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=128 time=10.3 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=128 time=6.85 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 6.851/9.834/12.417/1.985 ms

```
---
### Wazuh SIEM Manager Deployment
The Wazuh core stack was deployed using the official All-In-One(AIO) installation script, which compiles the indexer, Manager, and Dashboard services.

```bash
# Update local packages
sudo apt update && sudo apt upgrade -y

# Download and execute the automated Wazuh setup script
curl -sO [https://packages.wazuh.com/4.x/wazuh-install.sh](https://packages.wazuh.com/4.x/wazuh-install.sh)
sudo bash wazuh-install.sh -a
```
### Firewall Configuration (UFW)
```bash
# TCP Port 1514 (Agent log forwarding)
# TCP Port 1515 (Agent registration)
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw reload
```
### Wazuh Installation hasf finished
![Finished Installation](/images/finishedinstallation.png)
You need to save the login credentials;
### Wazuh Login
![Wazuh Login](/images/wazuhlogin.png)
### Wazuh Dashboard
![Wazuh Dashboard](/images/wazuhdashboard.png)
---

## Agent Installation & Enrollment
### Windows Server 2022 Agent
Deployed using an elevated administrative PowerShell session pointing back to the Wazuh Manager:
```bash
invoke-webrequest -Uri [https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.6-1.msi](https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.6-1.msi) -OutFile ${env:tmp}\wazuh-agent.msi

Start-Process -FilePath msiexec.exe -ArgumentList "/i ${env:tmp}\wazuh-agent.msi /q WAZUH_MANAGER='192.168.64.10' WAZUH_REGISTRATION_SERVER='192.168.64.10' WAZUH_ACTIVE_RESPONSE='yes'" -NoNewWindow -Wait

# Start the Agent Service
NET START WazuhSvc
```
### Agent Deployed Screenshot
![WindowsServer Agent Deployed](/images/WinServer01.png)
![WindowsServer Agent Deployed](/images/winserver02.png)

---
### Ubuntu Desktop Agent
Manually pulled the debian package, configured the registration variable, and enabled the agent service:

```bash
# Pull the specific Debian package
curl -so wazuh-agent.deb [https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.6-1_amd64.deb](https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.6-1_amd64.deb)

# Install and link to Manager IP
sudo WAZUH_MANAGER='192.168.64.10' dpkg -i wazuh-agent.deb

# Reload systemd and launch the daemon
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

### Ubuntu Desktop Agent Deployed
![Agent Deployed](/images/ubuntu01.png)
![Agent Deployed](/images/ubuntu02.png)
---

## Security Testing & Threat Detection
To validate that logs are parsed and flagged correctly, a simulated SSH Brute Force Attack was executed on the the Ubuntu Desktop node from the Wazuh Manager.

### Step 1: Attack Simulation
Initiating repeated connection attempts with a non-existent username (hacker) to trigger logging in /var/log/auth.log
```bash
ssh hacker@192.168.64.30
# Repeatedly inputted wrong credentials 3+ times
root@ubuntu:~# ssh hacker@192.168.64.30
hacker@192.168.64.30's password: 
Permission denied, please try again.
hacker@192.168.64.30's password: 
Permission denied, please try again.
hacker@192.168.64.30's password: 
hacker@192.168.64.30: Permission denied (publickey,password).
root@ubuntu:~# ssh hacker@192.168.64.30
hacker@192.168.64.30's password: 
Permission denied, please try again.
hacker@192.168.64.30's password: 
Permission denied, please try again.
hacker@192.168.64.30's password: 
```
Checking local auth logs on Ubuntu Desktop
```bash
user@ubuntu22-desktop:~$ sudo tail -n 10 /var/log/auth.log 
[sudo] password for user: 
Jul 14 19:42:29 ubuntu22-desktop sshd[59083]: Failed password for invalid user hacker from 192.168.64.10 port 53882 ssh2
Jul 14 19:42:30 ubuntu22-desktop sshd[59083]: Connection closed by invalid user hacker 192.168.64.10 port 53882 [preauth]
Jul 14 19:42:30 ubuntu22-desktop sshd[59083]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.64.10 
Jul 14 19:42:44 ubuntu22-desktop sshd[59085]: Invalid user hacker from 192.168.64.10 port 48048
Jul 14 19:42:47 ubuntu22-desktop sshd[59085]: pam_unix(sshd:auth): check pass; user unknown
Jul 14 19:42:47 ubuntu22-desktop sshd[59085]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.64.10 
Jul 14 19:42:49 ubuntu22-desktop sshd[59085]: Failed password for invalid user hacker from 192.168.64.10 port 48048 ssh2
Jul 14 19:42:51 ubuntu22-desktop sshd[59085]: pam_unix(sshd:auth): check pass; user unknown
Jul 14 19:42:53 ubuntu22-desktop sshd[59085]: Failed password for invalid user hacker from 192.168.64.10 port 48048 ssh2
Jul 14 19:44:03 ubuntu22-desktop sudo:     user : TTY=pts/1 ; PWD=/home/user ; USER=root ; COMMAND=/usr/bin/tail -n 10 /var/log/auth.log
```
---
### Step 2: SIEM Analysis
Navigating to the Wazuh Web UI (Discover module), we successfully queried and analyzed the alert genrated under the security signature:
* Rule ID: 5710
* Rule Description: sshd: Attempt to login using a non-existent user
* Triggered Severity Level: 5 (Medium Alert)
![SIEM Analysis](/images/5710.png)
---
## Conclusion
This project successfully demonstrates the workflow of setting up central log aggregation across distinct Operating System environments.
Through the implementation of a managed virtual network switch and the configurations of end-device agents, we achieved:
* Real-time endpoint monitoring
* Threat intelligence ingestion via standardized event logging.
* Precise security event identification  through custom/default Rule matching.