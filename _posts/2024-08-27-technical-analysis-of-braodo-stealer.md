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


![image](https://github.com/user-attachments/assets/e82a3e08-8070-4119-bd1c-bfb5470cf55f)


As we can see the Python Stealer is targeting various browsers for credential stealing, and the stealer is using Telegram bot for exfiltration of stolen credentials. So, let us check out the key functions of this stealer. 


#### Chrome Credentials Stealing

![image](https://github.com/user-attachments/assets/42966e64-9b38-491b-88a4-330c383a3d92)


As we can see on the script that this stealer is targeting Chrome browser by initially creating a folder known as `Chrome` and then it goes ahead and copies important and sensitive files such as Local State , Cookies and Login data and passes the data to a function known as `encrypt`. Now let us check the working of the encrypt function.

![image](https://github.com/user-attachments/assets/600b3531-ea7f-4378-aa70-ee457f00c76d)

![image](https://github.com/user-attachments/assets/00435315-5057-4f05-90ea-370a94d071b8)

![image](https://github.com/user-attachments/assets/e8748626-04d8-4bb6-bf03-6beb46fd3f61)



Initially the encrypt function goes ahead and loads the local state file which is basically a JSON file so it uses json.load function to parse the file and then decodes the key from the encrypt_key element using base64 decoding, further it decrypt the encrypted data or the data blob using `CryptUnprotectData` API. Then it goes ahead and saves the decrypted key into a file known as `master_key.txt` . Then it connects to SQLite database and then collect the cookies for Facebook and then stores the cookies to a file for further exfiltration.


#### Firefox Credentials Stealing


![image](https://github.com/user-attachments/assets/2f0bd959-f968-45a1-aa2b-6f028ffe05d3)

The stealer enumerates all the Firefox profiles, then it goes ahead and copies sensitive files such as `cookies.sqlite` , `key4.db` and `logins.json` and then it further copies all the data for encryption and decryption using the same function as above. 


#### Edge Credentials Stealing

![image](https://github.com/user-attachments/assets/57f2bd39-6aeb-43c2-8454-128eb14df9da)

The stealer enumerates all the profiles for Microsoft Edge browser and then it further goes and copies the `Cookies` , `Login data` , `Web data` , `Local State` into the directory known as `Edge` and then it goes ahead and further passes the data for encryption and decryption.


#### Other Browsers


![image](https://github.com/user-attachments/assets/19c9c335-4d6d-40ba-bda5-91a35ce6de50)

![image](https://github.com/user-attachments/assets/e75f9219-f564-4746-880a-21d1ddc6d8e5)

![image](https://github.com/user-attachments/assets/4a44eeff-e1d8-4476-883a-8a820a0647fb)


Similar to previous browsers the stealer also targets `Opera Browser` , `Brave Browser` and `Chromium Browser` where it creates a specific folder , copies sensitive data and further encrypts and decrypts it and saves it for exfiltration.


#### Miscellaneous


![image](https://github.com/user-attachments/assets/4a2788d8-0a0e-4193-b3cc-09d6f9f70ff7)

![image](https://github.com/user-attachments/assets/35c0362b-80df-43ac-b413-67863ac5ac34)

![image](https://github.com/user-attachments/assets/a767dc5a-65ea-41b4-8759-53881347989d)

Once it has completed stealing all the sensitive information and then it goes ahead and saves all the information in a ZIP file and exfiltrates the data over Telegram bot with unique identification of the victim such as name of the country , city , state along with user name and windows version attached for better identification of the victim.


## Features

- Browser Cookie stealing .
- Browser Password Stealing .
- Task Enumeration.
- Telegram based exfiltration.

## Conclusion

Thank you reader, for reading my blog, I have just started my journey with malware analysis, if you find something wrong please feel free to reach me out, thank you once again for reading.










