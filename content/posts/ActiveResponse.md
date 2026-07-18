---
title: "Lab 5: Automated Incident Response via Wazuh Active Response"
date: 2026-07-15
description: "Implementing automated threat mitigation (SOAR) in a virtualized SOC environment. Simulating a malicious SSH brute force attack using Kali Linux and automatically blocking the threat actor via OSSEC and IPtables."
tags: ["Wazuh", "Active Response", "SOAR", "Kali Linux", "Firewall", "IPtables", "Defensive Security"]
categories: ["Homelab Projects"]
showToc: true
TocOpen: false
---
<style>
  .max-w-prose, article, .prose { 
    max-width: 100% !important; 
    width: 100% !important; 
  }
</style>

## Executive Summary
Detecting a threat is only half the battle; responding to it efficiently is what minimizes the blast radius. This project demonstrates the deployment of **Automated Incident Response (SOAR capabilities)** using the Wazuh Active Response engine. 

By separating the attacker infrastructure onto a dedicated **Kali Linux** instance, we simulate an unauthenticated brute-force scenario. Upon detection, the Wazuh Manager dynamically instructs the target endpoint to drop all subsequent traffic from the threat actor's IP address using native system firewalls (`IPtables`).

---
## Phase 1: Dynamic Interaction Flow Diagram
```bash
[ Attacker: 192.168.64.10 ] ──(SSH Brute Force)──> [ Ubuntu Desktop: 192.168.64.30 ]
                                                           │
                                                   (Sends Event Log)
                                                           │
                                                           ▼
[ Wazuh Dashboard UI ] <──(Triggers Rule 5710)── [ Wazuh Manager: 192.168.64.10 ]
                                                           │
                                                 (Sends Block Command)
                                                           │
                                                           ▼
                                            [ Ubuntu Desktop: 192.168.64.30 ]
                                            (Firewall blocks Attacker IP via IPtables!)
```

---

```bash
root@ubuntu:~# nano /var/ossec/etc/ossec.conf

```
```bash
"Before making any changes, backing up the configuration file first is a best practice for any system administrator."
```
```bash
root@ubuntu:~# cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak
```
```bash
root@ubuntu:~# ls -l /var/ossec/etc/
total 80
-rw-r----- 1 root wazuh   179 Jul 14 19:57 client.keys
drwxrwx--- 2 root wazuh  4096 Jul 14 14:20 decoders
-rw-r----- 1 root wazuh 14961 Jun 26 03:01 internal_options.conf
drwxrwx--- 4 root wazuh  4096 Jul 14 14:20 lists
-rw-r----- 1 root wazuh   320 Jun 26 03:01 local_internal_options.conf
-rw-r----- 1 root wazuh   114 Jun 23 22:20 localtime
-rw-rw---- 1 root wazuh  9386 Jul 15 11:24 ossec.conf
-rw-r----- 1 root root   9386 Jul 15 11:30 ossec.conf.bak

```
## Phase 2: Active Response Block
Put the following xml codes under active-response tags
```bash
 <command>
    <name>firewall-drop</name>
    <executable>firewall-drop</executable>
    <timeout_allowed>yes</timeout_allowed>
</command>

```
#### Alert Rule 5710 တတ်လာရင် အဲဒီ Attacker IP ကို ၁၀ မိနစ် (စက္ကန့် ၆၀၀) block ခိုင်းခြင်း
```bash
<active-response>
    <command>firewall-drop</command>
    <location>local</location>
    <rules_id>5710</rules_id>
    <timeout>600</timeout>
</active-response>
```
After making changes reboot Wazuh Manager
```bash
root@ubuntu:~# systemctl restart wazuh-manager.service 

```
## Phase 3: Attack Simulation & Auto Block
Before I do the attack, I put a new Kali VM in the Topology.

```bash
[ Kali Linux Attacker ] ──(1. SSH Brute Force)──> [ Ubuntu Desktop Agent ]
  (192.168.64.99)                                      (192.168.64.30)
         │                                                    │
         │                                           (2. Ships /var/log/auth.log)
         │                                                    │
         ▼                                                    ▼
[ Attacker IP BLOCKED! ] <──(4. Runs firewall-drop)── [ Wazuh Manager Server ]
 (Blocked for 10 Mins via                             (192.168.64.10)
   Ubuntu Agent iptables)                                     │
                                                     (3. Triggers Rule 5710)
                                                              │
                                                              ▼
                                                     [ Wazuh Dashboard UI ]
                                                    (Visualizes Level 5 Alert)
```
### Adding New Kali Machine
![Adding Kali Machine](/images/add-kali.png)
Wazuh Manager ကို restart လုပ်ပြီးတဲ့နောက်မှာ Active Response စမ်းသပ်ပါမယ်။
Before we do attacking try to ping to the victim pc.
```bash
┌──(kali㉿kali)-[~]
└─$ ping 192.168.64.30 -c 4
PING 192.168.64.30 (192.168.64.30) 56(84) bytes of data.
64 bytes from 192.168.64.30: icmp_seq=1 ttl=64 time=5.48 ms
64 bytes from 192.168.64.30: icmp_seq=2 ttl=64 time=2.27 ms
64 bytes from 192.168.64.30: icmp_seq=3 ttl=64 time=2.00 ms
64 bytes from 192.168.64.30: icmp_seq=4 ttl=64 time=2.24 ms

--- 192.168.64.30 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 1.998/2.998/5.483/1.438 ms


```
### SSH Attack Simulation
```bash
┌──(kali㉿kali)-[~]
└─$ sudo ssh hacker@192.168.64.30
[sudo] password for kali: 
The authenticity of host '192.168.64.30 (192.168.64.30)' can't be established.
ED25519 key fingerprint is: SHA256:ogzqcZTNzWW8GZ+rwFnwc0ou1MNeduszOYKNTFceoE8
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.64.30' (ED25519) to the list of known hosts.
hacker@192.168.64.30's password: 
Connection closed by 192.168.64.30 port 22
```
### Ping Failed
```bash
┌──(kali㉿kali)-[~]
└─$ sudo ssh hacker@192.168.64.30
^C
                                                                                          
┌──(kali㉿kali)-[~]
└─$ sudo ssh hacker@192.168.64.30
^C
                                                                                          
┌──(kali㉿kali)-[~]
└─$ ping 192.168.64.30 -c 4
PING 192.168.64.30 (192.168.64.30) 56(84) bytes of data.

--- 192.168.64.30 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3049ms

```
### Screenshot of Ping Failed
![Ping Failed](/images/ping-didn'twork.png)

### Finding Alerts on Wazuh Manager
Type the following in the DQL Filter
```bash
rule.id: 5710 AND agent.name: "ubuntu22-desktop"
``` 
![Finding Alerts](/images/alertsfinding.png)

### Finding Alerts Automatically Blocked by Active Response
#### Filtered by Firewall Dropped
```bash
firewall-drop
```
![Dropped by Firewall](/images/firewall-drop.png)
#### Filtered by Rule ID 5710
```bash
rule.id: 5710
```
![Filtered by ID](/images/ruleid-5710.png)

#### Checking logs on Ubuntu Desktop
```bash
user@ubuntu22-desktop:~$ sudo tail -n 10 /var/ossec/logs/active-responses.log
[sudo] password for user: 
2026/07/15 14:11:45 active-response/bin/firewall-drop: {"version":1,"origin":{"name":"node01","module":"wazuh-execd"},"command":"add","parameters":{"extra_args":[],"alert":{"timestamp":"2026-07-15T04:11:45.168+0000","rule":{"level":5,"description":"sshd: Attempt to login using a non-existent user","id":"5710","mitre":{"id":["T1110.001","T1021.004"],"tactic":["Credential Access","Lateral Movement"],"technique":["Password Guessing","SSH"]},"firedtimes":1,"mail":false,"groups":["syslog","sshd","authentication_failed","invalid_login"],"gdpr":["IV_35.7.d","IV_32.2"],"gpg13":["7.1"],"hipaa":["164.312.b"],"nist_800_53":["AU.14","AC.7","AU.6"],"pci_dss":["10.2.4","10.2.5","10.6.1"],"tsc":["CC6.1","CC6.8","CC7.2","CC7.3"]},"agent":{"id":"002","name":"ubuntu22-desktop","ip":"192.168.64.30"},"manager":{"name":"ubuntu"},"id":"1784088705.121831","full_log":"Jul 15 04:11:43 ubuntu22-desktop sshd[11501]: Invalid user hacker from 192.168.64.144 port 42412","predecoder":{"program_name":"sshd","timestamp":"Jul 15 04:11:43","hostname":"ubuntu22-desktop"},"decoder":{"parent":"sshd","name":"sshd"},"data":{"srcip":"192.168.64.144","srcport":"42412","srcuser":"hacker"},"location":"journald"},"program":"active-response/bin/firewall-drop"}}
```
## Conclusion
This project demonstrates a fully functional Closed-Loop Incident Response system. By abstracting the attacking agent from the control tier, I managed to cleanly isolate and drop malicious network interaction without disrupting the availability or integrity of core management routes.

This successfully concludes Lab 2. The architecture is now natively fortified to drop fast-moving credential attack vectors in real-time