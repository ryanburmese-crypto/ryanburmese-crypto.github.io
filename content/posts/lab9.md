---
title: "Lab 9: Building a Local AI News Automation Agent with Docker, Ollama, n8n & Telegram"
description: "Step-by-step guide to building a privacy-focused local AI news digestion pipeline using Docker, Ollama (Llama 3), n8n workflow automation, and Telegram bot integration."
date: '2026-07-28T14:25:50+10:00'
tags: ["Docker", "Ollama", "n8n", "Telegram", "AI Agent", "Ubuntu", "Automation", "Cybersecurity"]
layout: "page"
---
<style>
  .max-w-prose, article, .prose { 
    max-width: 100% !important; 
    width: 100% !important; 
  }
</style>

## Executive Summary
This lab demonstrates how to build an automated, self-hosted AI news digestion pipeline using Docker, Ollama, n8n, and Telegram on Ubuntu 22.04 LTS. The primary objective is to replace manual website searching with an intelligent agent that fetches cybersecurity and IT news from RSS feeds, summarizes the key details locally using an open-source Large Language Model (Llama 3), and automatically delivers structured daily updates to a personal Telegram chat. By hosting all AI processes locally via Ollama, this solution ensures data privacy, reduces operational API costs, and optimizes daily security monitoring workflows.

Technical Overview & Architecture


```bash
[RSS Feed Source] 
       │
       ▼
[n8n Automation Node] ──(Schedule Trigger)──► [Ollama Local LLM (Llama 3)]
       │                                                   │
       ▼                                                   ▼
[Format Payload] ◄───────────────────────────── [Structured Summary]
       │
       ▼
[Telegram Bot Node] ──► [Telegram Mobile/Desktop Client]
```

**Prerequisites and Environment**
* Operating System: Ubuntu 22.04 LTS Desktop
* Core TEchnologies: Docker Engine, Docker Compose Plugin, Ollama, n8n Automation Engine.
* Third-Party Services: Telegram API (BotFather & Chat ID Verification).

# Step-by-Step Implementation
## Step 1: Install Docker & Docker Compose


## Update System Package
```bash
ryan@burman:~$ sudo apt update && sudo apt upgrade -y
[sudo] password for ryan: 
Hit:1 http://au.archive.ubuntu.com/ubuntu jammy InRelease
Hit:2 http://au.archive.ubuntu.com/ubuntu jammy-updates InRelease              
Hit:3 http://au.archive.ubuntu.com/ubuntu jammy-backports InRelease 
```
**Add the official Docker GPG key and APT repository**
```bash
ryan@burman:~$ sudo apt install -y ca-certificates curl gnupg
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ca-certificates is already the newest version (20260601~22.04.1).
ca-certificates set to manually installed.
curl is already the newest version (7.81.0-1ubuntu1.25).
gnupg is already the newest version (2.2.27-3ubuntu2.5).
gnupg set to manually installed.
0 to upgrade, 0 to newly install, 0 to remove and 0 not to upgrade.
```
## Add the official Docker GPG key and APT repository
```bash
ryan@burman:~$ sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

## Adding Docker Repository to APT Sources

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  

 ryan@burman:~$ sudo apt update
Get:1 https://download.docker.com/linux/ubuntu jammy InRelease [48.5 kB]
Hit:2 http://au.archive.ubuntu.com/ubuntu jammy InRelease                      
Hit:3 http://au.archive.ubuntu.com/ubuntu jammy-updates InRelease              
Hit:4 http://au.archive.ubuntu.com/ubuntu jammy-backports InRelease            
Get:5 https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages [83.5 kB]
Hit:6 https://downloads.claude.ai/claude-desktop/apt/stable stable InRelease   
Hit:7 https://dl.google.com/linux/chrome-stable/deb stable InRelease           
Hit:8 http://security.ubuntu.com/ubuntu jammy-security InRelease
Fetched 132 kB in 1s (106 kB/s)                        
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.

```
## Updating Repository and Installing Docker & Compose Plugin
```bash

ryan@burman:~$ sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  docker-ce-rootless-extras pigz
```
## Verify active installation versions
```bash
ryan@burman:~$ docker --version
Docker version 29.6.2, build dfc4efb
ryan@burman:~$ docker compose version
Docker Compose version v5.3.1

```

# Step 2: Create Docker Compose for n8n & Ollama

## Creating Lab Directory
```bash

ryan@burman:~$ mkdir -p ~/homelab/n8n-ai-agent
ryan@burman:~$ cd ~/homelab/n8n-ai-agent/
ryan@burman:~/homelab/n8n-ai-agent$

# Creating docker-compose.yml
ryan@burman:~/homelab/n8n-ai-agent$ nano docker-compose.yml

```

## Code for docker-compose.yml
```javascript
version: '3.8'

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama_ai
    restart: always
    ports:
      - "11434:11434"
    volumes:
      - ollama_storage:/root/.ollama

  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n_automation
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - WEBHOOK_URL=http://localhost:5678/
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  ollama_storage:
  n8n_data:
```
## Screenshot
![docker-compose.yml](/images/lab9/image.png)
---
![docker-compose.yml](/images/lab9/image1.png)
---

## Starting Docker
```bash
ryan@burman:~/homelab/n8n-ai-agent$ sudo docker compose up -d
WARN[0000] /home/ryan/homelab/n8n-ai-agent/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
[+] up 23/23
 ✔ Image docker.n8n.io/n8nio/n8n:latest Pulled                                                    179.6s
 ✔ Image ollama/ollama:latest           Pulled                                                    564.9s
 ✔ Network n8n-ai-agent_default         Created                                                     0.1s
 ✔ Volume n8n-ai-agent_ollama_storage   Created                                                     0.0s
 ✔ Volume n8n-ai-agent_n8n_data         Created                                                     0.0s
 ✔ Container ollama_ai                  Started                                                     1.2s
 ✔ Container n8n_automation             Started                                                     1.3s
ryan@burman:~/homelab/n8n-ai-agent$ 

```
# Step 3: Download Local AI Model into Ollama

```bash

ryan@burman:~/homelab/n8n-ai-agent$ sudo docker exec -it ollama_ai ollama run llama3
pulling manifest 
pulling 6a0746a1ec1a: 100% ▕██████████████████████████████████████████▏ 4.7 GB                         
pulling 4fa551d4f938: 100% ▕██████████████████████████████████████████▏  12 KB                         
pulling 8ab4849b038c: 100% ▕██████████████████████████████████████████▏  254 B                         
pulling 577073ffcc6c: 100% ▕██████████████████████████████████████████▏  110 B                         
pulling 3f8eb4da87fa: 100% ▕██████████████████████████████████████████▏  485 B                         
verifying sha256 digest 
writing manifest 
success 

```
# Step 4: Create Telegram Bot
- Find @BotFather in your Telegram Desktop.
- ./newbot and hit `Enter`
- Keep the HTTP API Token from BotFather
- send /start

```bash
Done! Congratulations on your new bot. You will find it at t.me/ryanburmese_ai_bot. You can now add a description, about section and profile picture for your bot, see /help for a list of commands. By the way, when you've finished creating your cool bot, ping our Bot Support if you want a better username for it. Just make sure the bot is fully operational before you do this.

Use this token to access the HTTP API:
7872134958:AACIUcNPF_eWv7#####################

Keep your token secure and store it safely, it can be used by anyone to control your bot.

For a description of the Bot API, see this page: https://core.telegram.org/bots/api
```
## Screenshot of creating Bot via Telegram
![BotFather](/images/lab9/image2.png)

**Find @userinfobot and send a message to get the `Chat ID`  and keep your ID**
```bash
Hi Welcome To @userInfo3Bot 🖐

📚 Help : /help

🔔 Bot News : @PyteTeam

👤 Your ID : 65xxxxxxx
```
**Ollama**
![Ollama Setup](/images/lab9/image3.png)
---
![Ollama Setup](/images/lab9/image4.png)
---
![Ollama Setup](/images/lab9/image5.png)
---
![Ollama Setup](/images/lab9/image6.png)
---
![Ollama Setup](/images/lab9/image7.png)
---
![Ollama Setup](/images/lab9/image8.png)
---
![Ollama Setup](/images/lab9/image9.png)

## Conclusion

This lab successfully built a working prototype for a privacy-focused local AI news agent. Key accomplishments include:

* **Complete Infrastructure Decoupling:** By hosting Ollama and n8n inside Docker containers, all data processing occurs within local system boundaries, removing reliance on costly external API services like OpenAI.
* **Automated Threat Intelligence:** Automated fetch-summarize-dispatch routines significantly reduce manual research overhead for cybersecurity and system engineering professionals.
* **Real-time Endpoint Delivery:** Telegram integration ensures critical zero-day notifications and system security news are delivered straight to mobile or desktop devices immediately.