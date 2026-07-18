---
title: "Lab 1: Building a Nested Virtualization Lab with EVE-NG on Ubuntu"
date: 2026-07-12
tags: ["Virtualization", "Ubuntu", "EVE-NG", "VMware"]
categories: ["Infrastructure"]
summary: "How to resolve performance issues and enable hardware acceleration in a nested hypervisor environment."
---
<style>
  .max-w-prose, article, .prose { 
    max-width: 100% !important; 
    width: 100% !important; 
  }
</style>
## Introduction
In this lab, I successfully built a nested virtualization environment using **VMware Workstation on Ubuntu 22.04**, hosting an **EVE-NG** instance.

## Technical Challenges & Fixes
* **The Problem:** Virtual machines inside EVE-NG (especially Windows Server nodes) were frequently freezing and performing extremely poorly.
* **The Root Cause:** Hardware acceleration wasn't being passed down to the inner QEMU/KVM hypervisor layer.
* **The Solution:** 1. Shut down the EVE-NG VM in VMware Workstation.
  2. Navigated to **Processor Settings** and enabled **"Virtualize Intel VT-x/EPT or AMD-V/RVI"**.
  3. Verified host capability via Terminal: `cat /sys/module/kvm_intel/parameters/nested` (Expected output: `Y` or `1`).

## Outcome
Enabling nested virtualization completely resolved the kernel freezing issues, allowing heavy Windows Server nodes to boot and query database operations smoothly.