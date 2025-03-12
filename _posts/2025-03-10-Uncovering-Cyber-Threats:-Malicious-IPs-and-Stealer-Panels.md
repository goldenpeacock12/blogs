---
title:  "Fresh Infections Of Braodo Stealer With Updated Techniques"
layout: post
categories: malware-analysis
---

## Contents

- Introduction
- Connections of the Malicious IPs
  - Identified Malicious IPs
  - Activity and Behavior of IPs
    - **Analysis of IP `152.32.139.38`**
    - **Analysis of IP `140.245.30.252`** 
- Stealer Panels Analysis
  - Panel 1 **-** PoisonX Stealer
  - Panel 2 - Amatera Stealer
  - Panel 3 - Lethalvoid
- Conclusion.

## Introduction

Hello everyone! This time, I decided to explore **network hunting** and analyze some **malicious IPs, stealer panels, and a suspicious website**.
I found these IPs through **Twitter** and conducted a basic investigation using tools like **VirusTotal, Validin, FOFA, and Shodan**. This was my first time working with **network hunting** and using these tools, so it was more of a **learning experience** for me.
In this blog, I’ll share my findings and insights from this research. Let’s dive in!

## Connections of the Malicious IPs

### Identified Malicious IPs

I discovered two **malicious IPs** from **Twitter**: **`152.32.139.38`** and **`140.245.30.252`**. To verify their nature, I searched for them on **VirusTotal**, which confirmed that these IPs were flagged as **malicious**.
Next, I used **FOFA** to gather more details about these IPs, such as the **connected domains** and **hosting locations**. I will explain my findings in the sections below.

### Activity and Behavior of IPs

#### **Analysis of IP 152.32.139.38**

  While analyzing **IP 152.32.139.38**, I found a connected **domain** with the title **"Bad Request!"**. Additionally, all other linked domains displayed            **"Million OK !!!!"** in their body content.
  Curious about this pattern, I specifically searched for **IPs returning "Million OK" in their response body**. This led me to **two ASNs** associated with the IP:

  - **ASN 135377 (Korea)**
  - **ASN 32097 (US)**
    
