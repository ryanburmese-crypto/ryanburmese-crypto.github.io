---
title: "Lab 12: Building a Custom Kali Linux Pentester Toolbox Container"
description: "Step-by-step guide to building a portable, lightweight Kali Linux Docker container equipped with real-world reconnaissance, web application security, credential testing, and post-exploitation tools."
date: '2026-08-10T14:10:00+10:00'
tags: ["Docker", "Kali Linux", "Cybersecurity", "Pentesting", "Red Team", "Nmap", "Metasploit", "DevSecOps"]
layout: "page"
---
<style>
  .max-w-prose, article, .prose { 
    max-width: 100% !important; 
    width: 100% !important; 
  }
</style>

## Executive Summary
This lab demonstrates how to build a custom, lightweight, and portable Kali Linux Pentester Toolbox Container using Docker. Instead of deploying heavy Kali Linux virtual machines or installing security utilities directly on target hosts, penetration testers can quickly launch this container in any environment (Cloud, On-Premises, or Pivot Nodes) for reconnaissance, vulnerability analysis, web app assessment, and network security testing.

## Lab Objectives
By completing this lab, you will be able to:

 * Build a custom Kali Linux Docker image using a Dockerfile.

 * Package popular penetration testing and cybersecurity tools into a single container.

 * Configure Docker network settings and privileges for advanced scanning tools (like `nmap`).

 * Manage wordlists and path dependencies for web testing utilities (like `ffuf`).

 * Mount local host directories to save scan results permanently.

 * Run a lightweight, portable security testing environment anywhere without using heavy virtual machines.

## Architecture & Tool Categories
```bash
                     Security Tester / Red Teamer
                                    │
                                    ▼
                    Kali Pentester Toolbox Container
                                    │
    ┌──────────────────┬────────────┴──────────┬──────────────────┐
    ▼                  ▼                       ▼                  ▼
Recon & OSINT    Network & Scanners      Web Security      Post-Exploit/Cracking
  • amass          • nmap                  • burpsuite       • john
  • subfinder      • masscan               • sqlmap          • hashcat
  • httpx          • netcat                • ffuf / wfuzz    • hydra
  • dnsrecon       • wireshark/tshark      • nikto           • impacket
```

### Step 1: Create a Project Directory
First, open the terminal and create a folder to hold the project files.
```bash
system@admin:~$ sudo mkdir -p ~/homelab/pentest-toolbox
system@admin:~$ cd ~/homelab/pentest-toolbox/
system@admin:~/homelab/pentest-toolbox$ 
```
### Step 2: Create the Dockerfile
A `Dockerfile` tells Docker how to build the custom image.
1. Open a new file name `Dockerfile` using nano:

```bash
system@admin:~/homelab/pentest-toolbox$ sudo nano Dockerfile
```
2. Paste the following configuration into the file.
```yaml
FROM kalilinux/kali-rolling:latest

# Environment Variable to prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Update system repositories and install real-world Pentesting tools
RUN apt-get update && apt-get install -y --no-install-recommends \
    # Core Utilities & Networking
    curl \
    wget \
    git \
    vim \
    tmux \
    iputils-ping \
    net-tools \
    iproute2 \
    dnsutils \
    jq \
    python3 \
    python3-pip \
    python3-venv \
    ca-certificates \
    #
    # Network Discovery & Port Scanning
    nmap \
    masscan \
    netcat-openbsd \
    tshark \
    tcpdump \
    traceroute \
    #
    # Reconnaissance & OSINT
    amass \
    subfinder \
    httpx-toolkit \
    dnsrecon \
    whois \
    #
    # Web Application Security & Fuzzing
    sqlmap \
    ffuf \
    wfuzz \
    nikto \
    gobuster \
    #
    # Password Cracking & Credential Attacks
    john \
    hashcat \
    hydra \
    wordlists \
    #
    # Active Directory & Post-Exploitation / Pivoting
    python3-impacket \
    smbclient \
    chisel \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Set SecLists directory path or alias for convenience
WORKDIR /root/tools

# Set default command to interactive shell
CMD ["/bin/bash"]
```
### Step 3: Build the Docker Image
Run the following command to build the container image:
```bash
system@admin:~/homelab/pentest-toolbox$ sudo docker build -t kali-pentest-toolbox:v1 .
```
Note: The first build may take a few minutes depending on your internet connection speed.

**Expected Output**
```bash
[+] Building 277.7s (7/7) FINISHED                                                                  docker:default
 => [internal] load build definition from Dockerfile                                                          0.0s
 => => transferring dockerfile: 1.33kB                                                                        0.0s
 => [internal] load metadata for docker.io/kalilinux/kali-rolling:latest                                      4.2s
 => [internal] load .dockerignore                                                                             0.0s
 => => transferring context: 2B                                                                               0.0s
 => [1/3] FROM docker.io/kalilinux/kali-rolling:latest@sha256:f49124869e4eee549315879c3bd7ef92f23b8753fec75  12.5s
 => => resolve docker.io/kalilinux/kali-rolling:latest@sha256:f49124869e4eee549315879c3bd7ef92f23b8753fec754  0.0s
 => => sha256:8ada140629654c49961d11c5878c1e052f2707a5a0c60f510cd778455881087d 52.22MB / 52.22MB             11.8s
 => => extracting sha256:8ada140629654c49961d11c5878c1e052f2707a5a0c60f510cd778455881087d                     0.7s
 => [2/3] RUN apt-get update && apt-get install -y --no-install-recommends     curl     wget     git     v  213.8s
 => [3/3] WORKDIR /root/tools   
  => exporting to image                                                                                       47.0s 
 => => exporting layers                                                                                      38.0s 
 => => exporting manifest sha256:38ce164a2b39ebd9a6c25931411bebecfa33f681a9c3453f3e934523c291231a             0.0s 
 => => exporting config sha256:14e6cb759c5c4f334f664a718d2a7f56f88f2c6274f6aec8f84c7e263978f63f               0.0s 
 => => exporting attestation manifest sha256:8e68278b76c83a44ffb32f9767a171c13f372735194808e50b6b0f0b5e1e1e7  0.0s 
 => => exporting manifest list sha256:ac57bc3478e4e657cc503194b187aefbb4699cc298c41e0ce846e6b227dd0776        0.0s
 => => naming to docker.io/library/kali-pentest-toolbox:v1                                                    0.0s
 => => unpacking to docker.io/library/kali-pentest-toolbox:v1 
```
**To confirm the image was created sucessfully:**
```bash
system@admin:~/homelab/pentest-toolbox$ docker images
```
**Expected Output**
```bash
                                                                                               i Info →   U  In Use
IMAGE                            ID             DISK USAGE   CONTENT SIZE   EXTRA
admin-toolbox:v1                 837ec9da992c        550MB          149MB    U   
docker.n8n.io/n8nio/n8n:latest   769d3a624534       2.48GB          386MB    U   
kali-pentest-toolbox:v1          ac57bc3478e4       2.65GB          729MB        
ollama/ollama:latest             4dea9fb51194       8.04GB         3.26GB    U   

```
### Step 4: Run the Container
Launch the new pentesting container using the following command:
```bash
system@admin:~/homelab/pentest-toolbox$ docker run -it --rm --privileged --net=host -v ~/homelab/pentest-toolbox/workspace:/root/workspace kali-pentest-toolbox:v1
┌──(root㉿admin)-[~/tools]
└─# 

```
**What the flags mean:**
* `-it` : Opens an interactive terminal session.
* `--rm` : Automatically deletes the container when you exit.
* `--privileged` : Gives full access rights to network network interces.
* `--net=host` : Uses your main computer's network directly (best for scanning - for educational purpose only). 
* `-v` : Connects a folder on your computer to `/root/workspace` inside the container so scan results are saved permanently.

### Step 5: Test The Pentest Tools
Once inside the container, we can test various pentesting utilities:

**1. Network Scan (Nmap)** 
Run a basic network scan against your local host.
```bash
nmap -sC -sV -oA /root/workspace/scan_results 127.0.0.1
```
**Expected Output**
```bash
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-10 04:29 +0000
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000020s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
631/tcp  open  ipp     CUPS 2.4
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Home - CUPS 2.4.7
|_http-server-header: CUPS/2.4 IPP/2.1
5678/tcp open  rrac?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Accept-Ranges: bytes
|     Cache-Control: public, max-age=86400
|     Last-Modified: Mon, 10 Aug 2026 04:12:19 GMT
|     ETag: W/"5e28-19fe9df184a"
|     Content-Type: text/html; charset=utf-8

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.49 seconds
```
```bash
┌──(root㉿admin)-[~/workspace]
└─# ls
scan_results.gnmap  scan_results.nmap  scan_results.xml

┌──(root㉿admin)-[~/workspace]
└─# cat scan_results.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nmaprun>
<?xml-stylesheet href="file:///usr/share/nmap/nmap.xsl" type="text/xsl"?>
<!-- Nmap 7.99 scan initiated Mon Aug 10 04:29:36 2026 as: /usr/lib/nmap/nmap -sC -sV -oA /root/workspace/scan_results 127.0.0.1 -->
<nmaprun scanner="nmap" args="/usr/lib/nmap/nmap -sC -sV -oA /root/workspace/scan_results 127.0.0.1" start="1786336176" startstr="Mon Aug 10 04:29:36 2026" version="7.99" xmloutputversion="1.05">
<scaninfo type="syn" protocol="tcp" numservices="1000" services="1,3-4,6-7,9,13,17,19-26,30,32-33,37,42-43,49,53,70,79-85,88-90,99-100,106,109-111,113,119,125,135,139,143-144,146,161,163,179,199,211-212,222,254-256,259,264,280,301,306,311,340,366,389,406-407,416-417,425,427,443-445,458,464-465,481,497,500,512-515,524,541,543-545,548,554-555,563,587,593,616-617,625,631,636,646,648,666-668,683,687,691,700,705,711,714,720,722,726,749,765,777,783,787,800-801,808,843,873,880,888,898,900-903,911-912,981,987,990,992-993,995,999-1002,1007,
```

**2. Subdomain Search (Subfinder)**

Search for active subdomains of a target domain:

```bash
┌──(root㉿admin)-[~/tools]
└─# subfinder -d google.com

               __    _____           __         
   _______  __/ /_  / __(_)___  ____/ /__  _____
  / ___/ / / / __ \/ /_/ / __ \/ __  / _ \/ ___/
 (__  ) /_/ / /_/ / __/ / / / / /_/ /  __/ /    
/____/\__,_/_.___/_/ /_/_/ /_/\__,_/\___/_/

		projectdiscovery.io

[INF] Current subfinder version v2.14.0 (outdated)
[INF] Loading provider config from /root/.config/subfinder/provider-config.yaml
[INF] Enumerating subdomains for google.com

[INF] Found 76627 subdomains for google.com in 1 minute 22 seconds


```
**Web Directory Fuzzing (ffuf)**

Scan a target web server for hidden directories. In this lab I tested the real working website of mine, and I masquerade the real one. 
```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://target.local/FUZZ
```
I got the following error message;
```bash
Encountered error(s): 1 errors occurred.
	* stat /usr/share/wordlists/dirb/common.txt: no such file or directory

Fuzz Faster U Fool - v2.1.0-dev

```
I installed the wordlists and seclists to my container.
```bash
┌──(root㉿admin)-[~/tools]
└─# apt-get update&& apt-get install -y wordlists seclists

```
**Expected Output**
```bash
Get:1 http://kali.download/kali kali-rolling InRelease [34.0 kB]
Get:2 http://kali.download/kali kali-rolling/contrib amd64 Packages [103 kB]
Get:3 http://kali.download/kali kali-rolling/non-free-firmware amd64 Packages [15.8 kB]
Get:4 http://kali.download/kali kali-rolling/non-free amd64 Packages [168 kB]
Get:5 http://kali.download/kali kali-rolling/main amd64 Packages [21.4 MB]
Fetched 21.7 MB in 6s (3464 kB/s)                                                                                 
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
wordlists is already the newest version (2026.2.0).
Solving dependencies... Done
The following NEW packages will be installed:
  seclists
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 545 MB of archives.
After this operation, 1935 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 seclists all 2025.3-0kali1 [545 MB]
31% [1 seclists 210 MB/545 MB 38%]                                                                   6818 kB/s 49s^Fetched 545 MB in 1min 23s (6553 kB/s)                                                                            
Selecting previously unselected package seclists.
(Reading database ... 27480 files and directories currently installed.)
Preparing to unpack .../seclists_2025.3-0kali1_all.deb ...
Unpacking seclists (2025.3-0kali1) ...
Setting up seclists (2025.3-0kali1) ...
Processing triggers for wordlists (2026.2.0) ...
/var/lib/dpkg/info/wordlists.postinst: 37: unzip: not found

```
**Extract Wordlists**

Default wordlists in Kali Linux (like rockyou.txt) are often zipped using .gz compression, so I need to extract them before use. 
To exract the `rockyou` wordlist
```bash
gunzip /usr/share/wordlists/rockyou.txt.gz 2>/dev/null || true
```
**Fixing the `File Not Found` Error

When running `ffuf` again, an error popped up:
```bash
Encountered error(s): 1 errors occurred.
	* stat /usr/share/wordlists/dirb/common.txt: no such file or directo
```
Why this happened: The `dirb` wordlist is either missing or stored in a different location because Kali Linux did not include the `dirb` package by default.

**Step-by-Step Fix**
**1. Check what wordlist are currently installed**

Run the following command to inspect the contents of the `/usr/share/wordlists` folder:

```bash
└─# ls -la /usr/share/wordlists/
```
**Output**
```bash
total 136656
drwxr-xr-x 1 root root      4096 Aug 10 04:47 .
drwxr-xr-x 1 root root      4096 Aug 10 04:45 ..
lrwxrwxrwx 1 root root        28 Aug 10 04:45 john.lst -> /usr/share/john/password.lst
lrwxrwxrwx 1 root root        41 Aug 10 04:45 nmap.lst -> /usr/share/nmap/nselib/data/passwords.lst
-rw-r--r-- 1 root root 139921507 May  7 10:16 rockyou.txt
lrwxrwxrwx 1 root root        19 Aug 10 04:45 seclists -> /usr/share/seclists
lrwxrwxrwx 1 root root        25 Aug 10 04:45 wfuzz -> /usr/share/wfuzz/wordlist

```
**Result:** The `dirb` folder is missing:

**2. Install DIRB** 
```cmd
# apt-get update && apt-get install -y dirb
```
```bash
Hit:1 http://http.kali.org/kali kali-rolling InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Solving dependencies... Done
The following NEW packages will be installed:
  dirb
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 208 kB of archives.
After this operatgeion, 1511 kB of additional disk space will be used.
Get:1 http://http.kali.org/kali kali-rolling/main amd64 dirb amd64 2.22+dfsg-7 [208 kB]
Fetched 208 kB in 2s (88.5 kB/s)
Selecting previously unselected package dirb.

```
**3. Run `ffuf` again**

Now that `dirb` is installed. tried to run `ffuf` command again.
```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.local/FUZZ
# I run the real website of mine instead of target.local for the testing purpose.
```
```bash
┌──(root㉿admin)-[~/tools]
└─# ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.lcoal/FUZZ

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://target.local/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.svn/entries            [Status: 403, Size: 10410, Words: 2604, Lines: 213, Duration: 703ms]
.git/HEAD               [Status: 403, Size: 10407, Words: 2604, Lines: 213, Duration: 702ms]
.cvsignore              [Status: 403, Size: 10408, Words: 2604, Lines: 213, Duration: 706ms]
_baks                   [Status: 403, Size: 10403, Words: 2604, Lines: 213, Duration: 705ms]
.mysql_history          [Status: 403, Size: 10412, Words: 2604, Lines: 213, Duration: 707ms]
_catalogs               [Status: 403, Size: 10407, Words: 2604, Lines: 213, Duration: 705ms]
.subversion             [Status: 403, Size: 10409, Words: 2604, Lines: 213, Duration: 707ms]
.profile                [Status: 403, Size: 10406, Words: 2604, Lines: 213, Duration: 709ms]
.bashrc                 [Status: 403, Size: 10405, Words: 2604, Lines: 213, Duration: 708ms]
:: Progress: [4614/4614] :: Job [1/1] :: 16 req/sec :: Duration: [0:00:44] :: Errors: 1 ::

```

**Web Vulnerability Scan (SQLMap)**
```bash
┌──(root㉿admin)-[~/tools]
└─# sqlmap -u "https://target.local/index.php?id=1" --batch
```
```bash
  ___
       __H__
 ___ ___[.]_____ ___ ___  {1.10.6#stable}
|_ -| . [.]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program
[*] starting @ 02:55:01 /2026-08-11/

[02:55:01] [INFO] testing connection to the target URL
[02:55:03] [WARNING] potential permission problems detected ('Access is forbidden')
[02:55:03] [WARNING] the web server responded with an HTTP error code (403) which could interfere with the results of the tests
[02:55:03] [INFO] checking if the target is protected by some kind of WAF/IPS
[02:55:04] [WARNING] reflective value(s) found and filtering out
[02:55:04] [INFO] testing if the target URL content is stable
[02:56:27] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[02:56:38] [WARNING] GET parameter 'id' does not seem to be injectable
[02:56:38] [CRITICAL] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. Please retry with the switch '--text-only' (along with --technique=BU) as this case looks like a perfect candidate (low textual content along with inability of comparison engine to detect at least one dynamic parameter). If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'
[02:56:38] [WARNING] HTTP error codes detected during run:
403 (Forbidden) - 87 times

[*] ending @ 02:56:38 /2026-08-11/
```
### Lessons Learned

- **Container Portability:** Packaging security tools inside a Docker container makes them easy to run anywhere without losing settings or needing re-installation.  
- **Network Permissions:** Running advanced tools like `nmap` inside Docker requires special flags (such as `--privileged` or `--net=host`) to access raw network sockets properly.
- **Wordlist Management:** Default Kali Linux images do not include all wordlists. Installing packages like `dirb` or `seclists` is necessary for web fuzzing tools like `ffuf`.
- **Data Persistence:** Using Docker volume mounting (`-v`) ensures that scan results and logs are saved permanently on the host machine even after the container is deleted. 

## # Conclusion

Building a custom Kali Linux Pentester Toolbox container provides a fast, reliable, and standardized environment for security testing. By combining Docker with popular cybersecurity utilities, administrators and penetration testers can troubleshoot networks and perform security audits efficiently without the overhead of traditional virtual machines.