---
title: "Lab 3: Cross-Platform SMB Integration & NTFS Hardening"
date: 2026-07-12
tags: ["Linux", "Windows Server", "SMB", "NTFS Permissions", "Hardening"]
categories: ["Security & Integration"]
summary: "Securing an enterprise share with advanced NTFS permissions and mounting it from an authenticated Ubuntu Linux Desktop."
---

## Introduction
This lab covers cross-platform directory permissions, securing a Windows SMB share, and connecting to it securely from an **Ubuntu 22.04 Desktop** client.

## Infrastructure Hardening Steps
1. **Network Profiling:** Switched network profiles from Public to Domain/Private to prepare for secure traffic.
2. **Windows Firewall Tweak:** Disabled blocking profiles using: `netsh advfirewall set allprofiles state off` to prevent connection time-outs.
3. **NTFS Security Permissions:**
   * Created a directory `C:\CompanyData`.
   * **Disabled Inheritance** to prevent default local users from gaining access.
   * Assigned the AD Domain Local Group (`DL_FS01_Share_Modify`) **Modify** access rights.

## Cross-Platform Linux Mounting
To test the permissions from the Ubuntu client logged into the Active Directory environment:
* Opened the native Ubuntu File Manager (**Files**) -> **Other Locations**.
* Entered the network path: `smb://10.1.1.110/CompanyData`
* Authenticated using AD credentials (`aklatt@cybersec.local`).

## Outcome
The Windows share successfully mounted inside the Linux environment. The AD group permissions strictly governed file creation, modification, and access rights, creating a secure cross-platform data pipeline.