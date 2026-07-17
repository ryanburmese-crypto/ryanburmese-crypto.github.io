+++
title = 'Building My Lab Environment with VMware Workstation'
date = '2026-07-12T12:23:12+10:00'
draft = false
tags = ["homelab", "linux", "virtualization", "cybersecurity"]
categories = ["System Engineering"]
showToc = false
description = "A deep dive into how I set up my personal Linux and cybersecurity home lab using VMware Workstation."
+++

Hi everyone! Welcome to my tech blog. In this first post, I want to share how I set up my hands-on practice environment for Linux system administration and cybersecurity monitoring.

## Why a Home Lab?

As an aspiring system engineer, having a safe sandbox to break things and fix them is crucial. A well-structured home lab allows me to test enterprise distributions, configure firewalls, and analyze logs without risking any production environment.

## My Current Lab Infrastructure

I am currently utilizing **VMware Workstation** as my primary type-2 hypervisor. Here is a quick breakdown of my setup:

* **Gateway/Firewall:** pfSense / GNS3 for network segmentation.
* **Server Nodes:** AlmaLinux and Rocky Linux for enterprise-grade service hosting.
* **Security Monitoring:** A dedicated SIEM instance to analyze logs and system events.

### Network Topology

To keep things organized, I divided my virtual network into three main zones:
1. **Management LAN:** For SSH and administrative access.
2. **Production/Service Zone:** Where active web servers and services reside.
3. **Isolated Testing Zone:** For malware analysis and risky configuration tests.

---

## Key Challenges Overcome

During the setup on my Linux host, I encountered a few kernel module compilation issues with VMware Workstation. Specifically, handling version compatibility required cloning the latest modules from the master branch to patch the host network components successfully. 

> **Tip:** If you see a `pathspec` error while building VMware modules on newer kernels, always double-check that you are fetching from the correct `master` upstream branch!

## Next Steps

Moving forward, I plan to integrate **Sigma rules** and automate some configuration playbooks using Ansible. Stay tuned for the next post where I will detail my AlmaLinux server hardening guide!