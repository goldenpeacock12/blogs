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

My motivation is to explore different areas of malware analysis and gain a deeper understanding of how cyber threats operate. By researching various APT groups, I can learn about different initial access techniques used by attackers. This helps in understanding the diverse ways cybercriminals infiltrate systems. The more APT groups I analyze, the better I can recognize attack patterns and improve threat detection skills. My goal is to continuously expand my knowledge in Malware Analysis and Threat Research.

## Technical Analysis

I wanted to learn about different APT groups and how they execute attacks in the initial phase. So, I collected three malware samples of different APT groups from MalwareBazaar and studied their initial attack methods.

Each group uses a different approach:

- SideWinder delivers a malicious document (Maldoc).
- Kimsuky uses a LNK file.
- Gamaredon deploys an HTA file.

Now, I will analyze these files to understand their attack techniques.

### Case 1 : Sidewinder using malicious document to drop RTF Payload.

During my analysis of an initial phishing attempt, I came across a document file. To examine its internal structure, I used the oleid command, which helped identify that the file contained an external relationship. This indicated that the document might be referencing or attempting to fetch external content.

![image](https://github.com/user-attachments/assets/e758e58b-4572-4da6-9d70-94862601927d)


To dig deeper, I used the `oleobj` command, which revealed that the document was configured to **download an RTF file from an external source**. The URL of this external file was embedded within the document, and the file being fetched was named **profile.rtf**. This suggests that the document was likely being used to execute a second-stage payload, making it a potential **malicious downloader**.

`hxxps://pubad-gov-lk.org-co.net/10472857/Profile.rtf`

Then, after doing some research, I found that this is a **decoy RTF file**, while the actual **RTF payload** had already been removed from the **C2 server**.

For anyone interested in researching further on **how SideWinder targets victims or delivers the actual RTF payload**, you can refer to this [research](https://blog.strikeready.com/blog/rattling-the-cage-of-a-sidewinder/).

