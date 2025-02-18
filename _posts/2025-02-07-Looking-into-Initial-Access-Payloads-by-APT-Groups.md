---
layout: post
title: "Looking into Initial Access Payloads by APT Groups"
date: 2025-02-07
categories: ["Non-PE Malware"]
---

- Introduction.
- Motivation.
- Technical Analysis
    - Case 1 : Sidewinder using malicious document to drop RTF Payload.
    - Case 2 : Kimsuky using malicious LNK file to drop PowerShell payload.
    - Case 3 : Gamaredon using HTA to drop further malware.
- YARA Rule.
    - Detecting Maldoc.
    - Detecting LNK.
    - Detecting HTA.
- IOCs.
- Limitations.
- MITRE ATT&CK.
