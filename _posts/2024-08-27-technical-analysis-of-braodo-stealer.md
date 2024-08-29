---
title:  "Technical Analysis of Braodo Stealer"
layout: post
categories: malware-analysis
---

## Contents

- Background.
- Technical Analysis.
    - Metadata
    - First stager analysis.
    - Second stager analysis.
- Features
- Conclusion.



## Background

Hello readers, I am a fresh grad  and a newbie and recently I am trying to learn malware analysis, so I picked up a stealer which is very simple known as Braodo Stealer, then I tried to analyze it. It consists of two stages, in the first stage we analyze a batch script  and in the second stage we understand how the python stealer steals the data and sends it to the telegram channel. It steals various  sensitive information like Passwords,Cookies  from Chrome browser, Opera Browser, Microsoft Edge. I found out the sample from Twitter and the publisher was [Yogesh Londhe.](https://x.com/suyog41/status/1815663636913779127?s=48)  

## Technical Analysis

In this phase, we will cover the technical analysis of the Braodo Stealer.


### Metadata

SHA-256 : `d90093eee7254776177bfd5833aee65cdf8980344d20f8366ab4ab5879b69549`

File-Name : `Number of people and time for this vacation.bat`

SHA-256 : `641f2db9e9fb8255337672fb8da9226225fa8e393b651c7c7ebbb5b555d4b755`

File-Name : `sim.py`


### First stager analysis.

Once we started to analyze the sample post downloading we found out that first-stager is a  malicious batch script, let us now look into the malicious script and analyze it.

![image](https://github.com/user-attachments/assets/2e071bb9-4bc4-4abd-903e-7fff0f457d95)

We can see that the batch script is filled or obfuscated with garbage code and upon removing the garbage code, I found that the actual script is basically spawning PowerShell along with some arguments.

![image](https://github.com/user-attachments/assets/68172c7b-3458-4c01-bd1c-b99324f6f936)

So, upon cleaning we can see that the first PowerShell command is downloading the file `update1.bat`, which was then saved into Startup folder, I found out and was confused on why it is storing the batch script to Startup folder, it turns out that the script was running a python file everytime, when the computer boots.

![image](https://github.com/user-attachments/assets/57ab14fa-ef6b-4cba-853d-6e87b5e06f05)

The second command is downloading a ZIP file which can be seen as `145.zip`, this zip file is saved at the Public folder with a filename of `Document.zip`.

![image](https://github.com/user-attachments/assets/137ee7bf-f58e-452d-9a3e-3e1b01f0900d)

The third Powershell command is extracting the zip archive and saving it in the Document folder.

![image](https://github.com/user-attachments/assets/6b95c529-6564-49ed-bd8f-248a072f02ca)

The last PowerShell command is running the `sim.py` file,  using the python executable. 
Now, we know that a file known as `sim.py` is being run, we will extract the file from the ZIP and then, look into the malicious second-stager.

### Second stager analysis

![image](https://github.com/user-attachments/assets/fa1555f2-9c9e-4800-aab4-1f1597f4a807)









