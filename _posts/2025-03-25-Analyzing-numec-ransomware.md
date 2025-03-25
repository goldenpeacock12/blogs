---
title:  "Numec : A new ransomware in wild."
layout: post
categories: malware-analysis
---

## Contents

- Introduction
- Backstory
- Technical Analysis
  - Code Structure & Functions
    - Self-Saving in System32 & Evasion from Static Analysis
    - Anti-Debugging, Privilege Escalation & Execution Bypass
    - Interesting Functions
  - Script Execution Flow
- YARA Rule

# Introduction

Hii everyone! In my previous blog, I explored different stealer panels and how they function. This time, I’ll be analyzing a ransomware in depth. In this blog, I will study its complete code, understand how it works, and break down its different functions. We’ll explore its encryption methods, execution process, and other key components. So, let’s get started!

# Backstory

While browsing Twitter, I came across a ([post](https://x.com/nextronresearch/status/1903080863757152538)) mentioning a newly discovered PowerShell-based ransomware. The post included multiple samples, so I decided to analyze them further. Upon checking these samples on VirusTotal, I found that all of them were flagged as malicious.

Out of these, I selected one specific sample for a deeper analysis, which I will be covering in this blog. After examining its code, I identified it as **Numec Ransomware**, and in the following sections, I will break down its functionality in detail.

# Technical Analysis

To understand how Numec Ransomware operates, I analyzed its PowerShell script in detail. This section breaks down its execution flow, encryption mechanism, and various functions. Let's dive into the technical aspects of this ransomware.

## Code Structure & Functions

The ransomware script can be broken down into three key parts:

### Self-Saving in System32 & Evasion from Static Analysis

```bash
param (
    [switch]$HiddenRun = $false
)
$scriptBaseName = "SvcHost"
$randomSuffix = -join ((65..90) + (97..122) | Get-Random -Count 8 | % {[char]$_})
$scriptNewName = "$scriptBaseName-$randomSuffix.ps1"
$scriptNewPath = "$env:SYSTEMROOT\System32\$scriptNewName"
```

The script above generates a random filename with a fixed prefix (`SvcHost-randomprefix.ps1`) and saves itself in the System32 directory, which basically gives the attacker an idea to avoid detection and  every time it runs, it changes its filename so that security tools can't easily identify it using fixed patterns.

### **Anti-Debugging, Privilege Escalation & Execution Bypass**


![image](https://github.com/user-attachments/assets/e07bd3c7-37a0-4676-bf22-dfff85cc188b)


First, it checks if a debugger is attached and exits immediately to prevent analysis using runtime debugging tools such as WinDbg, x64dbg, or Visual Studio Debugger. This is a very basic anti-debugging technique used to evade dynamic analysis. It works by utilizing [System.Diagnostics.Debugger]::IsAttached, which determines whether the process is being debugged. If the property returns true, the script immediately executes Exit, terminating itself before any further code can be analyzed in an interactive debugging session.

Then this PowerShell script ensures that it runs with administrator privileges and in a hidden window by checking if the $HiddenRun variable is set. If not, it retrieves the script’s file path using $PSCommandPath and verifies whether it is running with administrative rights. 

If the script is not elevated, it constructs an argument list that includes -ExecutionPolicy Bypass -WindowStyle Hidden -File "$scriptPath" -HiddenRun" and uses Start-Process with the -Verb RunAs flag to restart itself with administrator privileges. If it is already running as an administrator, it simply restarts itself in a hidden window without requesting elevation again. 

Then the script **creates a scheduled task** to ensure it runs automatically at a specific time every day. It does this using the following lines of code:

```powershell

$action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-ExecutionPolicy Bypass -WindowStyle Hidden -File `"$PSCommandPath`" -HiddenRun"
$trigger = New-ScheduledTaskTrigger -Daily -At "3AM"
Register-ScheduledTask -TaskName "SystemUpdate" -Action $action -Trigger $trigger -RunLevel Highest -Force
```

At last it **creates an action** that executes the PowerShell script in **hidden mode**. It **sets a trigger** to run the script **daily at 3 AM**, ensuring it keeps running even after system reboots. It **registers the task** under the name `"SystemUpdate"`, making it look like a legitimate system update process. It runs the task with **highest privileges**. In the next section I will analyze the interesting functions.

### **Interesting Functions**

The Ransomware script contains additional functions responsible for **encryption, key generation, network communication and ransom note creation**.

```bash

function Generate-RandomPassword
function Get-EncryptedPassword
function Ensure-UltraPersistence
function Disable-SystemSleep
function Enable-SystemSleep
function Stop-InterferingProcesses
function Get-DriveList
function Disable-Defender
function Enable-Defender
function Spread-ToNetwork
function Encrypt-Drive
```

Each function plays a specific role in the execution and encryption process. We will go through these functions one by one to undrstand their purpose and how they contribute to the overall ransomware operation.

## Script Execution Flow

At the beginning of the **try block**, the script executes several functions, which seem to be preparing the system for encryption:

```powershell

Disable-SystemSleep
Stop-InterferingProcesses
Disable-Defender
```


![image](https://github.com/user-attachments/assets/fb98fa46-a25a-49d9-9547-e8f4ece82857)



**`Disable-SystemSleep`** prevents the system from entering sleep mode, ensuring the script runs without interruption.


![image](https://github.com/user-attachments/assets/06c73cd8-6dc1-44d2-b17e-e861b0e40e19)


The `Stop-InterferingProcesses` function terminates specific processes that may interfere with certain operations. It first defines an array of process names, including Windows Explorer, cloud storage services (OneDrive, Dropbox, Google Drive, iCloud Drive), and a backup service, which are commonly associated with file synchronization and system interactions.

``The function then iterates through this list, attempting to retrieve each process using Get-Process` while suppressing errors if the process is not found. If the process is running, it is forcibly terminated using Stop-Process -Force, ensuring that no prompts or warnings appear. By stopping these services, the script can disrupt backup and synchronization processes, potentially preventing file recovery or user interaction with the system.


![image](https://github.com/user-attachments/assets/56fb5606-ec83-42b6-8c49-754767f73821)



**`Disable-Defender`** disable Windows Defender, preventing it from blocking the malware.

Then the script first checks for available drives using the **`Get-DriveList`** function and 
if no drives are found, it **stops running** to avoid errors.

```bash
$drives = @(Get-DriveList)
if (-not $drives) {
    exit
}
```


![image](https://github.com/user-attachments/assets/7033bd39-c979-43c3-84bd-35ff1001a6ba)



These variables will track the number of drives, encrypted files, total time taken, and data size.

After obtaining the list of drives, the script continues with a series of actions aimed at **ensuring encryption, preventing recovery, and spreading itself across the network**.

```bash
$null = cmd /c "takeown /F `"C:\System Volume Information`" /D Y >nul 2>&1"
$null = cmd /c "icacls `"C:\System Volume Information`" /grant `"${env:USERNAME}:F`" /C /Q >nul 2>&1"
$criticalDirs = @("SystemRestore", "WindowsImageBackup", "SPP")
```

Then this script takes ownership of the "C:\System Volume Information" directory, which is normally restricted to the SYSTEM user, and grant full control to the currently logged-in user.

It does this by executing the `takeown` command via `cmd /c` to change ownership of the directory, ensuring that access is no longer restricted. Then, it uses icacls to modify permissions, granting full control  to the current user, allowing unrestricted modifications.

Both commands suppress output and errors. Finally, the script defines an array of critical system directories, including `SystemRestore`, `WindowsImageBackup`, and `SPP`, which are responsible for system recovery, backup images, and Software Protection Platform (SPP) files the script basically prepares for further actions such as disabling system restore, deleting backups, or preventing recovery mechanisms from functioning using these lines of code.

Windows **`Volume Shadow Copies`** help restore lost files. The script **removes all these backup copies** to prevent file recovery. The script then **goes through each detected drive** and locks (encrypts) the files using the `Encrypt-Drive` function . If encryption is successful, it **keeps track** of how many files were encrypted, the total encryption time, and the total size of encrypted files. The script **tries to spread itself** to other computers on the same network using the `Spread-ToNetwork` function . 

Now let us understand the work flow of `Encrypt-Drive` Function. The function starts by asking for a **drive letter** (e.g., `C:` or `D:`) where it will search for files to encrypt. It generates a **random AES encryption key with using function aes key** to lock the files securely. It has a **list of important file extensions** (`.docx`, `.xlsx`, `.jpg`, `.mp4`, `.zip`, etc.). Only these files will be encrypted, while system files remain untouched. 

```bash
$systemDirs = @("Windows", "Program Files", "Program Files (x86)", "System Volume Information", "Temp", "$Recycle.Bin", "ProgramData")
$excludePaths = $systemDirs | ForEach-Object { "$driveLetter\$_" }
```

It searches common user folders like **Documents, Desktop, Downloads, and Pictures**. If matching files are found, it starts **AES encryption** (Advanced Encryption Standard). 


![image](https://github.com/user-attachments/assets/339cd4f1-e3b7-4a30-bf1e-1caff210a97e)



The **original file is deleted** after encryption to prevent access to unencrypted data. The AES key is **converted to Base64** and then **encrypted using RSA encryption** (public key encryption). This **encrypted key is saved** on the desktop as `EncryptedKey.enc`. 

Now let us understand the work flow of `Spread-ToNetwork` Function .The function first **finds all shared folders** on the computer . After finding shared folders, it **copies the script into them**.

![image](https://github.com/user-attachments/assets/b0902e5e-4846-4b63-8571-b5d73f7d44ba)


![image](https://github.com/user-attachments/assets/d6c1200b-8ce1-4deb-b88a-188ee3767b1a)


![image](https://github.com/user-attachments/assets/2bfb57b9-8ffb-4c66-ac65-ec0bd21a936c)


The `Spread-ToNetwork` function heavily relies on **network shares** to propagate the script across a network. It first enumerates all shared folders on the system using `Get-WmiObject -Class Win32_Share`, filtering out only shares with `Type -eq 0`, which corresponds to disk-based shared folders. These shares are accessible over the network, allowing the script to copy itself into them. The function constructs the destination path using the format `\\\\$($share.__SERVER)\\$($share.Name)\\$scriptNewName`, which represents a UNC (Universal Naming Convention) path pointing to the shared folder on another system.

Additionally, it attempts to place the script inside the `StartUp` directory within the share (`$remoteStartup`), ensuring execution when the target machine restarts. If the file is successfully copied, the function invokes the WMI method `Win32_Process -Name Create` on the remote machine to execute the script in a hidden PowerShell window. To ensure persistence, it modifies the Windows registry (`SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run`) using remote registry access, setting an entry that launches the script at every startup.

Furthermore, the function attempts to spread via **hidden administrative shares** (`ADMIN$`), which exist on Windows systems for remote administration. It iterates through active IPs found in the ARP table (`arp -a`) and tries to copy the script to `\\\\$ip\\ADMIN$\\$scriptNewName`, following up with remote execution via WMI. This method exploits misconfigured or weakly secured network shares, enabling lateral movement between systems. If successful, the script gains execution privileges across multiple machines in the network, mimicking self-replicating malware techniques.


![image](https://github.com/user-attachments/assets/14262285-9fe1-4e92-a9fd-85c7d63dc920)


Now, looking into the function `Ensure-UltraPersistence` we can see that this script performs a lot of functions such as  modifying registry keys, creating a scheduled task, running a hidden service, and using Alternate Data Streams (ADS).

It does it by first copying itself to a new path to avoid deletion. Then, it manipulates Windows Registry keys in `HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` and `HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`, which ensures execution at startup. It randomizes registry entry names (SysUpdateCheck, WindowsEnhancer, etc.) to evade detection. Next, it modifies the Winlogon Shell key, ensuring that PowerShell runs alongside explorer.exe when a user logs in. The script also creates a Windows service (`WindowsUpdateSvcXXXX`) that automatically executes the script with a hidden PowerShell window. Next, it registers a scheduled task (`SystemBootExec`) with the highest privileges to execute the script.

Lastly, it leverages Alternate Data Streams (ADS) by embedding the script into `systemprofile:evilstream.ps1`, allowing execution while remaining hidden from traditional file listings.

![image](https://github.com/user-attachments/assets/75f40b72-a217-44a5-bc1b-bc0cdfb81dfb)


Looking into the later and the last part of the script it makes something which known as a watchdog script that runs on infinite loop, this script checks every 5 seconds whether the primary script (`$scriptNewPath`) exists and is running. If not, it restores the script from `$adsPath` and executes it in a hidden PowerShell process. The watchdog script is then saved in the system's temporary directory (`$SYSTEMROOT\Temp`) with a randomized name and executed with bypassed execution policies.

Next we will move ahead to the last part of this Ransomware

![image](https://github.com/user-attachments/assets/7e476267-c8a4-4750-83bf-e300e23ecbf7)


![image](https://github.com/user-attachments/assets/678b065e-36e8-4637-8259-82b6801b0e2f)


Finally, we can see that the functions `Enable-Defender`  & `Enable-SystemSleep`  performs the reverse decision which was basically disabled earlier at the beginning.

![image](https://github.com/user-attachments/assets/393a2b08-431a-43d9-ba1c-dc0c82e1103b)


So, finally we can see that, once the ransomware has finished performing all the tasks, it goes ahead and prints the ransom note into a file known as `GetFilesBack.txt`  , inside which the ransom note mentions the encryption summary and how to recover your files.

# YARA Rule

```bash
rule detect_numec_ransomware
{
	meta:
		author = "Priya"
		description = "This is the detection for numec ransomware"
		
	strings:
		$extension = ".numec"
		$ransomnote = "GetFilesBack.txt"
		$hklmpath = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
		$command = "takeown /F 'C:\System Volume Information' /D Y >nul 2>&1"
		$command2 = "powershell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -File"
		$dis_defender = "Set-MpPreference"
		$login_action = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
		$debugger_check = "System.Diagnostics.Debugger"
		
	condition:
		all of them
		
}

```

I am actually learning how to write YARA Rules, so please pardon me if the rule is not very upto the mark.

  
