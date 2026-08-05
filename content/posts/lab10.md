---
title: "Lab 10: Self-Hosted Automated AI Resume Tailoring Pipeline"
description: "Step-by-step guide to building a privacy-focused local AI resume and cover letter tailoring pipeline using Docker, Ollama (Llama 3), n8n workflow automation, and CloudConvert."
date: '2026-08-05T18:10:00+10:00'
tags: ["Docker", "Ollama", "n8n", "CloudConvert", "LLM", "AI Agent", "Ubuntu", "Automation"]
layout: "page"
---
<style>
  .max-w-prose, article, .prose { 
    max-width: 100% !important; 
    width: 100% !important; 
  }
</style>

## Executive Summary
This project demonstrates a self-hosted, privacy-focused automation pipeline built using Docker, n8n, Ollama (Llama 3), and CloudConvert. The solution automates the process of tailoring a candidate's master resume and generating a matching cover letter specific to any target job description.

By running Large Language Models (LLMs) locally via Ollama, sensitive personal data never leaves the host infrastructure during the generation process. The final output is automatically compiled into a clean, employer-ready Microsoft Word document (.docx) and saved to a local workspace.

## Technical Architecture & Workflow Design

```bash 
[ n8n Form Submission ]
          │
          ▼
[ Read Master Resume (.md) ]
          │
          ▼
[ HTTP Request (Ollama Llama 3) ]
          │
          ▼
[ Convert Text to File (.md) ]
          │
          ▼
[ CloudConvert (MD -> DOCX) ]
          │
          ▼
[ Save Final (.docx) to Disk ]
```

## Component Overview
* n8n Form Submission (Trigger): Captures the target company name and job description text from the user.
* Read File from Disk: Loads the local master_resume.md file from the host workspace.
* HTTP Request (Ollama API): Sends the master resume and job description to the local Ollama instance (llama3:latest) via API to generate a tailored resume and cover letter.
* Convert to File: Transforms the raw LLM text response into a structured .md binary file.
* Convert a File (CloudConvert): Calls the CloudConvert API to convert the Markdown file into a formatted Microsoft Word document (.docx).
* Write File to Disk: Writes the generated .docx file back to the host system using dynamic naming (e.g., tailored_resume_System Administrator _ Thales.docx).

## Infrastructure Setup & Installation Guide
### Prerequisites
* OS: Ubuntu 22.04 LTS / 24.04 LTS or Debian-based Linux environment
* Hardware: Minimum 8GB RAM (16GB recommended for LLM processing)
* Software: Docker Engine & Docker Compose Plugin installed

## Step 1: Docker & Docker Compose Setup
Update system package index and install required packages:

```bash
system@admin:~$ sudo apt update && sudo apt upgrade -y
sudo apt install -y ca-certificates curl gnupg lsb-release git
[sudo] password for system: 
Hit:1 http://au.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://au.archive.ubuntu.com/ubuntu noble-updates InRelease                                   
Hit:3 http://au.archive.ubuntu.com/ubuntu noble-backports InRelease                                          
Hit:4 https://dl.google.com/linux/chrome-stable/deb stable InRelease                                         
Hit:5 http://security.ubuntu.com/ubuntu noble-security InRelease                                    
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done

```
Add Docker’s official GPG key and repository:
```bash
system@admin:~$ sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
system@admin:~$ 
```
Install Docker Engine and Docker Compose plugin:
```bash
system@admin:~$ sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```
Enable and start Docker service, then add your current user to the docker group:
```bash
system@admin:~$ sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
Synchronizing state of docker.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable doc
```

### Directory Setup & docker-compose.yml Configuration
Create dedicated project directory structure:
```bash
system@admin:~$ mkdir -p ~/n8n-ollama-lab/workspace
cd ~/n8n-ollama-lab

```
Create a docker-compose.yml file:
```bash
system@admin:~/n8n-ollama-lab$ nano docker-compose.yml

```
Paste the following container orchestration manifest:

```yml
version: '3.8'

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    ports:
      - "11434:11434"
    volumes:
      - ollama_storage:/root/.ollama
    networks:
      - ai-network

  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_FUNCTION_ALLOW_BUILTIN=*
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
    volumes:
      - n8n_storage:/home/node/.n8n
      - ./workspace:/home/node/.n8n-files/workspace
    networks:
      - ai-network
    depends_on:
      - ollama

networks:
  ai-network:
    driver: bridge

volumes:
  ollama_storage:
  n8n_storage:
```
Launch services in detached mode:
```bash
system@admin:~/n8n-ollama-lab$ sudo docker compose up -d
WARN[0000] /home/system/n8n-ollama-lab/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
[+] up 16/18
 ✔ Image docker.n8n.io/n8nio/n8n:latest Pulled                                                                                   163.2s ✔ Image ollama/ollama:latest           Pulled                                                                                   574.0s
 ✔ Volume n8n-ollama-lab_ollama_storage Created                                                                                    0.0s
 ✔ Network n8n-ollama-lab_ai-network    Created                                                                                    0.0s
 ✔ Volume n8n-ollama-lab_n8n_storage    Created                                                                                    0.0s
 ✔ Container ollama                     Started                                                                                    0.5s
 ✔ Container n8n                        Started                                                                                    0.4s

```
Verify running containers:
```bash
ystem@admin:~/n8n-ollama-lab$ docker compose ps
WARN[0000] /home/system/n8n-ollama-lab/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
NAME      IMAGE                            COMMAND                  SERVICE   CREATED              STATUS              PORTS
n8n       docker.n8n.io/n8nio/n8n:latest   "tini -- /docker-ent…"   n8n       About a minute ago   Up About a minute   0.0.0.0:5678->5678/tcp, [::]:5678->5678/tcp
ollama    ollama/ollama:latest             "/bin/ollama serve"      ollama    About a minute ago   Up About a minute   0.0.0.0:11434->11434/tcp, [::]:11434->11434/tcp

```

---
### Step 2: Ollama Setup & Model Pulling
Pull the Llama 3 LLM model into the Ollama container:
```bash
system@admin:~/n8n-ollama-lab$ docker exec -it ollama ollama pull llama3
pulling manifest 
pulling 6a0746a1ec1a: 100% ▕██████████████████████████████████████████████████████████████████████████████████████████████████████████████ ▏ 4.7 GB/4.7 GB  6.9 MB/s      0s
verifying sha256 digest 
writing manifest 
success 

```
Verify available models:
```bash
system@admin:~/n8n-ollama-lab$ docker exec -it ollama ollama list
NAME             ID              SIZE      MODIFIED      
llama3:latest    365c0bd3c000    4.7 GB    2 minutes ago   
```
### Step 3: Workspace Preparation

Place your base Master Resume markdown file in the shared mounted workspace folder on the host:
```bash
system@admin:~/n8n-ollama-lab$ sudo nano ~/n8n-ollama-lab/workspace/master_resume.md
[sudo] password for system: 
system@admin:~/n8n-ollama-lab$ 

```
Add your complete background details into this file and save it.

Set proper file ownership and read permissions for n8n container user execution:

```bash
system@admin:~/n8n-ollama-lab$ sudo chmod -R 777 ~/n8n-ollama-lab/workspace/
```

### Step 4: Workflow Construction
* Open web browser and navigate to http://<SERVER_IP_OR_LOCALHOST>:5678.
* Complete owner account setup upon first load.
* Click Create Workflow and configure the nodes in sequence:
```bash
[ On form submission ] ---> [ Read File from Disk ] ---> [ HTTP Request ] ---> [ Convert to File ]  ---> [ Convert a File] ---> [Write File to Disk]
```
**n8n Setup**
![n8n Setup](/images/lab10/image5.png)

![n8n Setup](/images/lab10/image6.png)

#### Node 1: `On form submission` (Trigger)
* **Form Title:** `IT Job Tailor Assistant`
* **Form Fields:**
  * Field 1: `Target Company` (Text)
  * Field 2: `Job Description` (Textarea)


![n8n Setup](/images/lab10/image7.png)


![n8n Setup](/images/lab10/image9.png)

#### Node 2: `Read File from Disk` (`Read/Write Files from Disk`)
* **Operation:** `Read File(s) From Disk`
* **File Path:** `/home/node/.n8n-files/workspace/master_resume.md`
* **Output Property:** `data`

![n8n Setup](/images/lab10/image10.png)

#### Node 3: `HTTP Request` (Ollama Generation Node)
* **Method:** `POST`
* **URL:** `http://ollama:11434/api/generate`
* **Send Body:** `Enabled`
* **Body Content Type:** `JSON`
* **Specify Body:** `Using Fields Below`
* **Options -> Timeout:** `900000` (15 minutes limit to allow generation)

**Body Parameters:**


```t
| Parameter Name | Value Mode        | Value Content / Expression   | 
| :---           | :---              | :---                         |
| `model`        | Fixed             | `llama3:latest`              |
| `stream`       | Expression (`fx`) | `{{ false }}`                |
| `prompt`       | Expression (`fx`) | *(See Prompt Payload below)* |
```
**Prompt Payload:**
```bash
You are a highly detailed and expert IT Resume and Cover Letter Writer.

TASK:
Generate TWO distinct, high-quality, and complete Markdown documents:
1. # Tailored Resume
2. # Professional Cover Letter

CRITICAL RULES:
- Extrapolate and strictly use ALL real skills, experience, qualifications, and achievements from the MASTER RESUME TEXT below.
- NEVER use generic placeholders like [Your Name], [Company Name], [Dates], or [Manager Name]. Use exact details where available.
- Highlight technical strengths relevant to the job role.
- The Cover Letter must clearly express enthusiasm for the specific role described in the JOB DESCRIPTION, highlighting how the applicant's exact technical background maps to the position.

### JOB DESCRIPTION:
{{ $('On form submission').item.json['Job Description'] }}

### MASTER RESUME TEXT:
{{ Buffer.from($('Read/Write Files from Disk').item.binary.data.data, 'base64').toString('utf8') }}

Return ONLY the Markdown content for both sections without any introductory text or conversational preamble.
```
#### Node 4: `Convert to File`
* **Operation:** `Convert Text to File`
* **Text Input Field:** `response`
* **Put Output File in Field:** `data`
* **Options -> File Name:** `input.md`
* **Options -> Mime Type:** `text/markdown`

![n8n Setup](/images/lab10/image.png)

### Node 5: `Convert a file` (CloudConvert Node)
* **Resource:** `File`
* **Operation:** `Convert File`
* **Output Format:** `docx`
* **Binary Input Data:** `Enabled`
* **Binary Property:** `data`
* **Additional Options -> Input Format:** `md`

![n8n Setup](/images/lab10/image1.png)

#### Node 6: `Read/Write Files from Disk1` (Write File Node)
* **Operation:** `Write File to Disk`
* **File Path and Name (Expression `fx`):**
  ```bash
  {{ '/home/node/.n8n-files/workspace/tailored_resume_' + $('On form submission').item.json['Target Company'] + '.docx' }}
  ```
* **Input Binary Field:** `data`
![n8n Setup](/images/lab10/image11.png)
![n8n Setup](/images/lab10/image12.png)

### Constraints & Key Considerations

1. **Inference Latency & Timeouts:** Running LLMs locally (e.g., Llama 3 8B) on CPU hardware requires extending node HTTP timeout limits to `900000ms` (15 minutes) in n8n to avoid premature `ECONNABORTED` execution failures.
2. **Type Mapping in Payload Data:** The Ollama API strictly enforces Boolean data types for parameter parameters like `stream`. Sending `"false"` as a string causes Go unmarshaling errors; setting expression mode to `{{ false }}` ensures correct JSON payload formatting.
3. **MIME Type and Binary Input Definitions:** External conversion APIs require explicit declaration of input formatting (`Input Format: md`) when consuming binary streams directly from preceding nodes in memory.

#### Step 6: Testing & End-to-End Verification

1. Click **Execute Workflow** in n8n.
2. Open the Form URL generated by Node 1.
3. Fill in `Target Company` and paste a target `Job Description`.
4. Submit the form.
5. Check workflow logs in n8n to ensure all nodes complete with green checkmarks.
6. Verify output document generation on host system:

```bash
system@admin:~$ ls -la ~/n8n-ollama-lab/workspace/
total 20
drwxrwxrwx 2 system docker 4096 Aug  5 17:57  .
drwxrwxr-x 3 system docker 4096 Aug  5 12:19  ..
-rwxrwxrwx 1 root   root   3447 Aug  5 13:08  master_resume.md
-rw-r--r-- 1 system system 6563 Aug  5 17:55 'tailored_resume_System Administrator _ Thales.docx'

```
![n8n Setup](/images/lab10/image3.png)

![n8n Setup](/images/lab10/image4.png)
---
## Conclusion

This lab successfully demonstrates an end-to-end local AI workflow that takes a target job description, cross-references it with a candidate's master resume, and automatically outputs a fully formatted, customized `.docx` resume and cover letter[cite: 1]. The architecture ensures complete data privacy while streamlining the career application workflow through local infrastructure automation.