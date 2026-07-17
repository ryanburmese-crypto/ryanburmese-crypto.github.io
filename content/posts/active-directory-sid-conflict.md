---
title: "Lab 2: Active Directory Domain Services & Troubleshooting SID Conflicts"
date: 2026-07-12
tags: ["Active Directory", "Windows Server", "Sysprep", "Troubleshooting"]
categories: ["Active Directory"]
summary: "Resolving duplicate Security Identifier (SID) conflicts when joining cloned Windows Server nodes to an AD domain."
---

## Introduction
I deployed an Active Directory Domain Controller managing the `cybersec.local` domain. To expand the infrastructure, I set up a dedicated File Server (`FS01`).



## The SID Conflict Issue
When attempting to join the File Server to `cybersec.local`, the operation failed with the following critical error:
> *"The domain join cannot be completed because the SID of the domain you attempted to join was identical to the SID of this machine."*

### SID Issue Screenshot
![SID Issue on File Server](/images/sid-error-1.png)

This occurred because the File Server VM was a direct clone of the Domain Controller's base image, leading to identical Security Identifiers (SIDs).

## Remediation via Sysprep
To clean the machine's unique identifiers, I ran the Windows System Preparation tool on `FS01`:
1. Opened CMD as Administrator and executed: `C:\Windows\System32\Sysprep\sysprep.exe`
2. Selected **Enter System Out-of-Box Experience (OOBE)**.
3. Crucially ticked the **Generalize** checkbox to strip the old SID.
4. Set Shutdown options to **Restart**.

## Outcome
After restarting and re-configuring the network settings (IP: `10.1.1.110`, DNS: `10.1.1.100`), `FS01` successfully joined the `cybersec.local` domain with a unique SID, yielding the confirmation: *"Welcome to the cybersec.local domain."*

### SID Issue Solved Screenshot
![SID Issue Solved with sysprep](/images/sid-solved.png)
![](/images/sid-solved-1.png)