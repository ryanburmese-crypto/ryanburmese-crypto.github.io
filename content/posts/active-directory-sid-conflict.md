---
title: "Lab 2: Active Directory Domain Services & Troubleshooting SID Conflicts"
date: 2026-07-12
tags: ["Active Directory", "Windows Server", "Sysprep", "Troubleshooting"]
categories: ["Active Directory"]
summary: "How to fix duplicate Security Identifier (SID) errors when joining cloned Windows Server machines to an Active Directory domain."
---

<style>
  .max-w-prose, article, .prose { 
    max-width: 100% !important; 
    width: 100% !important; 
  }
</style>

## Introduction
I set up an Active Directory Domain Controller to manage the `cybersec.local` domain. To expand my network lab, I deployed a second Windows Server to act as a dedicated File Server (`FS01`).

## The SID Conflict Issue
When I tried to join the File Server to the `cybersec.local` domain, the process failed with this error message:

> *"The domain join cannot be completed because the SID of the domain you attempted to join was identical to the SID of this machine."*

### SID Issue Screenshot
![SID Issue on File Server](/images/sid-error-1.png)

This problem happened because I cloned the File Server VM directly from the Domain Controller's base image. As a result, both virtual machines shared the exact same Security Identifier (SID), which stops Active Directory from identifying them properly.

## Remediation via Sysprep
To generate a new, unique SID for the File Server, I used the Windows System Preparation (Sysprep) tool on `FS01`:

1. Opened Command Prompt (CMD) as an Administrator and ran: 
```cmd
C:\Windows\System32\Sysprep\sysprep.exe
```
2. Chose **Enter System Out-of-Box Experience (OOBE)** from the System Preparation Action menu.
3. Enabled the **Generalize** checkbox (this is the step that removes the old duplicate SID).
4. Set the Shutdown options to **Restart** and clicked OK.

## Outcome
Once the server restarted, I re-configured its network interfaces with the correct static profile (IP: `10.1.1.110`, DNS pointing to Domain Controller: `10.1.1.100`). 

With a fresh, unique SID generated, `FS01` successfully joined the `cybersec.local` domain and received the classic validation message: *"Welcome to the cybersec.local domain."*

### SID Issue Solved Screenshot
![SID Issue Solved with sysprep](/images/sid-solved.png)
![](/images/sid-solved-1.png)