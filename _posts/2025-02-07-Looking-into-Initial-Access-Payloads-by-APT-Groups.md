---
layout: post
title: "Looking into Initial Access Payloads by APT Groups"
date: 2025-02-07
categories: ["Non-PE Malware"]
---

## Contents
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

## Introduction
![image](https://github.com/user-attachments/assets/b975d5f5-9074-4bd4-b747-efbaaa380ffd)

I recently started learning about maldoc analysis as part of my malware analysis journey. During this process, I explored different APT groups, including SideWinder( 🇮🇳 ), Kimsuky ( 🇰🇵 ) , and Gamaredon ( 🇷🇺 ). I analyzed their samples, focusing on how they gain initial access. To conduct my research, I used platforms like VirusTotal, Twitter, and MalwareBazaar to find and study real-world malware samples.So, I am writing this blog which contains my findings for the same.

## Motivation

My motivation is to explore different areas of malware analysis and gain a deeper understanding of how cyber threats operate. By researching various APT groups, I can learn about different initial access techniques used by attackers. This helps in understanding the diverse ways cybercriminals infiltrate systems. The more APT groups I analyze, the better I can recognize attack patterns and improve threat detection skills. My goal is to continuously expand my knowledge in Malware Analysis Threat Research.
