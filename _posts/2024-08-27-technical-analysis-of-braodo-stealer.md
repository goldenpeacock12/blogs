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



