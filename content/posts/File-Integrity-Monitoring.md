---
title: "Lab 6: Real-Time File Integrity Monitoring (FIM) via Wazuh Syscheck"
date: 2026-07-15
description: "Implementing real-time File Integrity Monitoring (FIM) on a Windows Server 2022 endpoint using the Wazuh Syscheck engine to detect unauthorized file additions, deletions, and modifications."
tags: ["Wazuh", "FIM", "Syscheck", "Windows Server 2022", "SIEM", "Threat Hunting", "Defensive Security"]
categories: ["Homelab Projects"]
showToc: true
TocOpen: false
---
## Executive Summary
Maintaining data integrity across core infrastructure assets is vital for preventing advanced persistent threats (APTs) and unauthorized adjustments to configuration scripts. This project demonstrates the design and execution of **File Integrity Monitoring (FIM)** using the built-in Wazuh Syscheck engine. 

By target-monitoring a highly sensitive corporate directory inside a **Windows Server 2022 Standard** node, the SIEM architecture is structured to immediately intercept, calculate cryptographic hash shifts, and alert on system-level changes in real-time.

---

## Phase 1: Dynamic FIM Workflow Diagram
The dynamic interaction path below illustrates the telemetry processing sequence from the asset monitoring layer up to the centralized analytics suite:
```bash
[ Windows Server: 192.168.64.20 ] ──(Modifies C:\Important_Data\secret.txt)──> [ Syscheck Engine ]
                                                                                      │
                                                                             (Detects Hash Change)
                                                                                      │
                                                                                      ▼
[ Wazuh Dashboard UI ] <──(Triggers Rule 550 / 554)── [ Wazuh Manager: 192.168.64.10 ]
 (Displays File Modification Alert)
 ```
 ## Phase 2: Configuring FIM on Windows Server Agent
In Wazuh, File Integrity Monitoring is handled via the Syscheck daemon module. We must explicitly define the target folders and directories on the Windows asset that require state monitoring.

 ### Step 1: Creating a Target Directory on Windows Server
Log into the Windows Server virtual machine and create a new dedicated testing directory under the primary system partition root:

* Absolute Local Path: **C:\Important_Data**

* Inside this directory, provision a foundational plain text element named **secret.txt** to act as the baseline baseline asset.

 ![File Created](/images/ImportantData.png)

 ### Step 2: Accessing the Windows Wazuh Agent Configuration File
Before making any changes, backing up the configuration file first is a best practice for any system administrator. Open a command prompt or PowerShell instance with administrative rights and verify agent integrity rules, then open the primary configuration matrix using specialized text utilities:

```bash
C:\Program Files (x86)\ossec-agent\ossec.conf
```
![Open ossec.conf](/images/ossec.png)

### Step 3: Injecting continuous Monitoring Directives
Navigate inside the main <syscheck> configuration block element and append the target parameter path rule:
```bash
 <directories realtime="yes" check_all="yes">C:\Important_Data</directories>
```

![Adding Path to Check](/images/ossec-1.png)
Operational Parameter Rationale:
* realtime="yes": Instructs the OSSEC agent engine to bypass timed interval sweeps and immediately push alert logs to the manager within sub-seconds of detection.

* check_all="yes": Forces explicit evaluation mapping against MD5, SHA1, and SHA256 cryptographic check values, owner tags, and standard permission tables.

Commit the runtime changes to the running client engine by executing an elevated baseline restart inside PowerShell:

### Step 4: Restarting Wazuh Agent
```bash
PowerShell

Restart-Service -Name WazuhSvc
```
## Phase 3: File Modification Emulation & Ingestion

### Step 1: Modifing File Content (Emulating the Attack)
To emulate a malicious tampering attempt or unauthorized structural file insertion, an administrator or adversary account was used to open the protected target directory path **C:\Important_Data\secret.txt**.

The data contents inside the sample asset file were altered, adding unverified alphanumeric strings, and subsequently committed back to disk storage.

![Adding some content](/images/modified.png)

## Phase 4: SOC Threat Hunting & Intelligence Verification
Due to the persistent monitoring framework enabled by the agent configuration rules, the local endpoint calculated the file modification signature and pushed telemetry directly up to the aggregation server.

Dissecting Centralized Log Telemetry via DQL
Navigating to the main security monitoring panels, a query filter matching standard Syscheck file rules was applied inside the centralized search console window:
```bash
rule.id: 550 OR rule.id: 554
```
![Rule ID 550](/images/id5550.png)

### Alert Rule Mapping Matrix:
* `Rule 550`: Integrity checksum changed (Triggered when structural modifications alter baseline file hashes).

* `Rule 554`: File added to the system (Triggered when fresh files are dropped into protected segments).

### Forensic Ingestion Log Breakdowns
Evaluating the detailed metadata elements from the caught threat indicator reveals detailed host structural state tracking logs:

* Target File Path: `C:\Important_Data\secret.txt`

* File Size Shift Monitoring: syscheck.size_before: 215 bytes shifted directly to syscheck.size_after: 285 bytes following user injection.

* Cryptographic Preservation Check: Original data hash parameters (syscheck.md5_before: c0f508d69bfad230de86d766f104c520) were instantly flagged as broken, triggering immediate defensive awareness visibility on the platform dashboard.

🎯 Conclusion
By enforcing custom directory structures on the Windows instance, the SIEM environment successfully demonstrated closed-loop monitoring over file access operations. The deployment remains robustly configured to detect stealthy persistent mechanics, unauthorized payload uploads, or adjustments to configuration baselines.