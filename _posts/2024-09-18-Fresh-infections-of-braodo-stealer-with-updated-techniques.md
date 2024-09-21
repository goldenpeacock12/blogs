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
