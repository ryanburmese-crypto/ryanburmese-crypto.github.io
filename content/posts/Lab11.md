
---
title: "Lab 11: Building a Custom Linux System Administrator Toolbox Container"
description: "Step-by-step guide to building a custom Docker image packed with essential Linux networking, DNS, monitoring, and security tools for portable, containerized server troubleshooting."
date: '2026-08-05T20:29:44+10:00'
tags: ["Docker", "Ubuntu", "Linux", "SysAdmin", "DevOps", "Cybersecurity", "Networking", "Containerization"]
layout: "page"
---
<style>
  .max-w-prose, article, .prose { 
    max-width: 100% !important; 
    width: 100% !important; 
  }
</style>

## Executive Summary

This lab demonstrates how to build a custom Docker image designed specifically for Linux System Administrators. The container provides a portable troubleshooting and administration environment that includes commonly used networking, DNS, monitoring, security, and automation tools. Instead of installing utilities on multiple servers, administrators can launch a single container containing all essential tools required for diagnostics, troubleshooting, and infrastructure management.

## Lab Objectives

By completing this lab, I was able to:

- Build a custom Docker image from a Dockerfile
- Understand Docker image creation and management
- Create a reusable Linux administration toolkit
- Install network troubleshooting tools
- Install DNS diagnostic tools
- Install monitoring and performance analysis utilities
- Package administrative tools inside a portable container
- Run and manage custom Docker containers

## Architecture Overview

```bash
                     Administrator
                            |
                            v
              Docker Toolbox Container
                            |
    ------------------------------------------------
    |               |              |              |
    v               v              v              v
 Network Tools   DNS Tools   Monitoring Tools  Security Tools

 ping            dig          htop             openssl
 curl            host         btop             whois
 wget            nslookup     vmstat
 nmap                         iostat
```
## Environment
### Operating System
```bash
Ubuntu 24.04 LTS
```
### Technologies
```bash
Docker Engine
Dockerfile
Ubuntu Base Image
```

## Step 1: Create Project Directory
Create a dedicated working directory.
```bash
system@admin:~$ sudo mkdir -p ~/homelab/admin-toolbox
[sudo] password for system: 
system@admin:~$ cd ~/homelab/admin-toolbox/
system@admin:~/homelab/admin-toolbox$ 

```
Verify
```bash
pwd
```
**Expected Output**
```bash
system@admin:~/homelab/admin-toolbox$ pwd
/home/system/homelab/admin
```
### Step 2: Create Dockerfile
**Create a new Dockerfile**
```bash
system@admin:~/homelab/admin-toolbox$ sudo nano Dockerfile

```
**Insert the following content:**
```bash
FROM ubuntu:24.04

RUN apt update && apt install -y \
    curl \
    wget \
    iputils-ping \
    net-tools \
    dnsutils \
    nmap \
    netcat-openbsd \
    tcpdump \
    jq \
    git \
    python3 \
    openssl \
    traceroute \
    whois\
    sysstat

CMD ["/bin/bash"]
```
**Save and exit**

### Understanding the Dockerfile

**From**
```bash
FROM ubuntu:24.04
```
Defines the base operating system used by the container.

**RUN**
```bash
RUN apt update && apt install -y ...
```
Installs required Linux administration tools.

**CMD**
```bash
CMD ["/bin/bash"]
```

### Step 3: Build the Docker Image
**Build the impage using Docker.**

```bash
docker build -t admin-toolbox:v1 .
```
**Expected Output**

```bash
system@admin:~/homelab/admin-toolbox$ sudo docker build -t admin-toolbox:v1 .
[+] Building 56.6s (6/6) FINISHED                                                       docker:default
 => [internal] load build definition from Dockerfile                                              0.0s
 => => transferring dockerfile: 326B                                                              0.0s
 => [internal] load metadata for docker.io/library/ubuntu:24.04                                  10.4s
 => [internal] load .dockerignore                                                                 0.0s
 => => transferring context: 2B                                                                   0.0s
 => [1/2] FROM docker.io/library/ubuntu:24.04@sha256:561618e2c15bf2397621dd04f96926663a3b5616c18  5.5s
 => => resolve docker.io/library/ubuntu:24.04@sha256:561618e2c15bf2397621dd04f96926663a3b5616c18  0.0s
 => => sha256:966c395d29cb24a3faf7e04f32878fe5778819d4132daee4f47e2aaf7b9af924 29.75MB / 29.75MB  5.0s
 => => extracting sha256:966c395d29cb24a3faf7e04f32878fe5778819d4132daee4f47e2aaf7b9af924         0.4s
 => [2/2] RUN apt update && apt install -y     curl     wget     iputils-ping     net-tools      33.8s
 => exporting to image                                                                            6.9s 
 => => exporting layers                                                                           5.7s 
 => => exporting manifest sha256:70bea208e1f3d4554728fd8b4a6592ed84582fd9954700b928e447b7ba6215c  0.0s 
 => => exporting config sha256:e8e36600ebff64fd830481e33f8795fa33784824cd5a4e4827dbff7ed9c08ae3   0.0s 
 => => exporting attestation manifest sha256:1237517beac4311f1eafc16f73d854a1487045f0855dfb75dec  0.0s 
 => => exporting manifest list sha256:001cc6425bf4f32b77d61d65eec77a23b2944e091862bafd0718d4e74a  0.0s 
 => => naming to docker.io/library/admin-toolbox:v1                                               0.0s
 => => unpacking to docker.io/library/admin-toolbox:v1                                            1.2s
```
**Verify**
```bash
docker images
```

**Expected Output**
```bash
system@admin:~/homelab/admin-toolbox$ sudo docker images
                                                                                   i Info →   U  In Use
IMAGE                            ID             DISK USAGE   CONTENT SIZE   EXTRA
admin-toolbox:v1                 001cc6425bf4        503MB          138MB        
system@admin:~/homelab/admin-toolbox$ 

```
### Step 4: Launch the Container
**Start the toolbox container.**
```bash
system@admin:~/homelab/admin-toolbox$ sudo docker run -it admin-toolbox:v1
```
**It should now be inside the container**
```bash
root@61bfa44d1f67:/# 

```
### Step 5: Verify Installed Tools
**Ping Test**
```bash
ping -c 4 google.com
```
```bash
PING google.com (172.217.25.174) 56(84) bytes of data.
64 bytes from syd09s13-in-f14.1e100.net (172.217.25.174): icmp_seq=1 ttl=127 time=19.0 ms
64 bytes from syd09s13-in-f14.1e100.net (172.217.25.174): icmp_seq=2 ttl=127 time=21.6 ms
64 bytes from syd09s13-in-f14.1e100.net (172.217.25.174): icmp_seq=3 ttl=127 time=21.4 ms
64 bytes from syd09s13-in-f14.1e100.net (172.217.25.174): icmp_seq=4 ttl=127 time=21.7 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 19.015/20.948/21.740/1.121 ms
root@61bfa44d1f67:/# 

```
**DNS Lookup**
```bash
dig microsoft.com
```
```bash
; <<>> DiG 9.18.39-0ubuntu0.24.04.5-Ubuntu <<>> microsoft.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39967
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; MBZ: 0x0005, udp: 65494
;; QUESTION SECTION:
;microsoft.com.			IN	A

;; ANSWER SECTION:
microsoft.com.		5	IN	A	150.171.109.26

;; Query time: 9 msec
;; SERVER: 192.168.64.2#53(192.168.64.2) (UDP)
;; WHEN: Wed Aug 05 10:18:58 UTC 2026
;; MSG SIZE  rcvd: 58

```
**Host Command**
```bash
host github.com
```
```bash
github.com has address 4.237.22.38
github.com mail is handled by 0 github-com.mail.protection.outlook.com.

```
**Port Scan**
```bash
nmap localhost
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-08-05 10:20 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000010s latency).
Other addresses for localhost (not scanned): ::1
All 1000 scanned ports on localhost (127.0.0.1) are in ignored states.
Not shown: 1000 closed tcp ports (reset)

Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds

```
**HTTP Connectivity**
```bash
curl https://www.google.com
```
```bash
<!doctype html><html itemscope="
" itemtype="http://schema.org/WebPage" 
lang="en-AU"><head><meta content="text/html; charset=UTF-8" 
http-equiv="Content-Type"><meta content="/images/branding/googleg/1x/googleg_standard_color_128dp.png" 
itemprop="image"><title>Google</title>
<script nonce="TKoBPfqxz8I0vADHO8AEcQ">
(function(){var _g={kEI:'cw5zas-UIZfWkPIP98yEwAU',kEXPI:'0,203061,4036984,85211,9707,344796,226411,7,
5293906,12,12464,846,62039403,152393,65154,138360,37790,107648,41099,26230,25591,46339,32561,16179,2646,
24246,15888,3,1515,3355,6010,2,5999,5,14302,2,3105,5089,19,3007,22,774,6452,4,8172,12796,9199,5337,2891,2,13,8515,5280,
```
**Network Capture**
```bash
tcpdump -i eth0
```
```bash
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
10:23:01.961631 IP6 fe80::5009:79ff:fe7f:b706.mdns > ff02::fb.mdns: 0 [2q] PTR (QM)? _ipps._tcp.local. PTR (QM)? _ipp._tcp.local. (45)
10:23:02.041203 IP 61bfa44d1f67.37584 > 192.168.64.2.domain: 4490+ PTR? b.f.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.2.0.f.f.ip6.arpa. (90)
10:23:02.061675 IP 192.168.64.2.domain > 61bfa44d1f67.37584: 4490 NXDomain 0/1/0 (154)
10:23:02.062145 IP 61bfa44d1f67.40592 > 192.168.64.2.domain: 53539+ PTR? 6.0.7.b.f.7.e.f.f.f.9.7.9.0.0.5.0.0.0.0.0.0.0.0.0.0.0.0.0.8.e.f.ip6.arpa. (90)
10:23:07.067518 IP 61bfa44d1f67.40592 > 192.168.64.2.domain: 53539+ PTR? 6.0.7.b.f.7.e.f.f.f.9.7.9.0.0.5.0.0.0.0.0.0.0.0.0.0.0.0.0.8.e.f.ip6.arpa. (90)
10:23:07.176026 ARP, Request who-has burman tell 61bfa44d1f67, length 28
10:23:07.176033 ARP, Request who-has 61bfa44d1f67 tell burman, length 28
10:23:07.176047 ARP, Reply 61bfa44d1f67 is-at 0a:72:ac:8e:48:e5 (oui Unknown), length 28
10:23:07.176062 ARP, Reply burman is-at 52:09:79:7f:b7:06 (oui Unknown), length 28
10:23:12.072423 IP 61bfa44d1f67.39419 > 192.168.64.2.domain: 33685+ PTR? 2.64.168.192.in-addr.arpa. (43)
10:23:12.095704 IP 192.168.64.2.domain > 61bfa44d1f67.39419: 33685 NXDomain 0/1/0 (102)
10:23:12.096361 IP 61bfa44d1f67.36366 > 192.168.64.2.domain: 52806+ PTR? 1.0.17.172.in-addr.arpa. (41)
10:23:12.112655 IP 192.168.64.2.domain > 61bfa44d1f67.36366: 52806 2/0/0 PTR burman., PTR burman.local. (87)

```
**System Monitoring**
```bash
htop
```
![htop Screenshot](/images/lab11/image.png)

**JSON Analysis**
```bash
echo '{"name":"latt"}' | jq
```

```bash
root@61bfa44d1f67:/# echo '{"name": "admin"}' | jq
{
  "name": "admin"
}
root@61bfa44d1f67:/# 

```
### Step 6: Test Container Protability
Exit the container.
```bash
exit
```
```bash
root@61bfa44d1f67:/# exit
exit
system@admin:~/homelab/admin-toolbox$ 

```
**Run it again**
```bash
docker run -it admin-toolbox:v1
```

```bash
system@admin:~/homelab/admin-toolbox$ sudo docker run -it admin-toolbox:v1
root@0c3093f137c0:/# 

```
All tools remain available because they are packaged inside the Docker image.
---
### Skills Demonstrated

✅ Linux Administration

✅ Docker Image Creation

✅ Dockerfile Development

✅ Network Troubleshooting

✅ DNS Diagnostics

✅ Performance Monitoring

✅ Security Administration

✅ Infrastructure Troubleshooting

✅ Open Source Tool Integration

✅ Container Operations
---
### Real World Use Cases

This toolbox container can be used for:

- Network troubleshooting
- DNS diagnostics
- System monitoring
- Security investigations
- Infrastructure testing
- Linux server administration
- Container troubleshooting
- Training and lab environments
---
### Lessons Learned

- Docker images can package complete administrative environments.
- Custom images improve consistency across systems.
- Containerized toolkits eliminate repetitive software installation.
- Dockerfiles provide a reproducible method for building infrastructure tools.
- A portable administration toolkit can significantly improve troubleshooting efficiency.
---
### Conclusion

This lab successfully created a custom Linux System Administrator Toolbox container using Docker. By packaging commonly used administrative, networking, monitoring, and security tools into a single reusable image, infrastructure troubleshooting becomes faster, more portable, and more consistent across different environments. This project also reinforced Docker image creation, container management, and Linux administration fundamentals while providing a practical tool that can be reused in future homelab and production environments.