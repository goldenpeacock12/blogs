---
title:  "Fresh Infections Of Braodo Stealer With Updated Techniques"
layout: post
categories: malware-analysis
---

## Contents

- Background.
- Technical Analysis.
    - Metadata.
    - Encoding and variable rename.
    - Abusing Abobus-Obfuscator & Dropbox services.
    - Abusing .bmp file format.
- Conclusion.

## Background


Hi readers, hope you enjoyed my last blog on Braodo Stealer, as a newbie and as I'm just getting started I have decided to track the same, so recently I found that Braodo Stealer is back with a bunch of changes on their arsenal , which includes adding Base64 encoding along with renaming variable names and then abusing an open source obfuscator known as Abobas-Obfuscator. Well, the last but not the least that's using different file formats like .bmp and abusing Dropbox services for payload delivery.

## Technical Analysis.

In this phase, we will know different methods that this stealer uses to download the payload and run the malware code in the system

### Metadata

SHA-256 : `e314fc6308c31b600eb43fee5e4716034375f0a7e6916326c605ec955ead5757`
file name : batch_obfuscator.malwre

SHA-256 : `dba09794ee80639df8633b23219d29976bd7fabcb60db063023f3ba8f9cc1c9b`
file name : bmp_payload

SHA-256 : `a93d56fde501f5341c1c84e5a1f2f17253bbcada088898b2cb3df35c9ff68b93`
file name : duckyy26.sample

SHA-256 : `4d7892ce5812d01041cdeff95bc622dda3f6bb3add64045eb15ce73215080b52`
file name : Wukong.zip

### Encoding and variable rename.

### Abusing Abobus-Obfuscator & Dropbox services.
![abobus_obfuscator](https://github.com/user-attachments/assets/e0e1974e-1726-4b61-b109-54c0b6b6ed02)

### Abusing .bmp file format.

![bmp_payload](https://github.com/user-attachments/assets/1210ec36-f690-4c52-b7b8-39f1a72bd935)

## Conclusion.



