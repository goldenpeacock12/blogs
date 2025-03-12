---
title:  "Fresh Infections Of Braodo Stealer With Updated Techniques"
layout: post
categories: network-analysis
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

![blog_cover](https://github.com/user-attachments/assets/27222411-c859-45cc-8bdc-ec7f2db8b1d2)

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

  ![bl_1](https://github.com/user-attachments/assets/1a1dd64f-9ad3-474c-9097-812ba6cfc3db)

  While analyzing the **IP 152.32.139.38**, I found a **notable domain**: **o-r[.]kr**, which is significant because **".kr" domains are primarily associated with    South Korea**.

  Additionally, I discovered several other domains hosted under the **same ASN**, including:

  - **n-e[.]kr**
  - **p-e[.]kr**
  - **r-e[.]kr**
  - o-r[.]kr
  - sugarfungame[.]com

  ![bl_2](https://github.com/user-attachments/assets/a54fa542-ab6c-4545-ab4c-cf972f915e2e)

  Since these domains share the **same ASN** and exhibit **suspicious behavior**, they are suspected to be part of the same malicious infrastructure.
This indicates that the websites related to this IP are primarily **hosted in Korea and the US**.
Further analysis revealed that the **"Million OK" HTTP response** is commonly linked to the **Kimsuky APT group**, often uses **malicious domains, IPs, and compromised servers** to carry out attacks.

#### **Analysis of IP 140.245.30.252**

After analyzing this **IP**, I found that it is **hosted in the USA** under ASN 31898 . However, upon visiting the website associated with this IP, it displayed an **Indian website**.
Additionally, I observed that the site uses an **icon hash**:
**icon_hash = "-466135758"**

![bl_3](https://github.com/user-attachments/assets/f94fe9c0-618f-42b9-8d17-2b361991025c)

This specific hash value is known to represent something related to Indian government .

![bl_4](https://github.com/user-attachments/assets/042d078c-edc8-4cf4-86f0-6785dad0441c)

Through further research, I discovered that this **IP is malicious** and is associated with **APT36**, a **threat group known for cyber-espionage activities**.
APT36, also called **Transparent Tribe**, is a **Pakistan-linked APT group** primarily targeting **Indian government entities, defense organizations, and educational institutions**.

![bl_5](https://github.com/user-attachments/assets/db94f0c4-cd80-4858-b4ff-14838cabc4ae)

The IP address **140.245.30.252**, which is hosted in the **United States (ASN 31898)**, is actively communicating with multiple **Indian government websites** such as **data.gov.in, india.gov.in, esuvidha.gov.in, gem.gov.in**, and others. So, it stands out that APT36 has been using this malicious infrastructure for targeting Indian government .
Therefore, in a previous examples I explored the malicious infrastructures of Kimsuky and APT36.
Now in the next section I will focus on stealers.

## Stealer Panels Analysis

I found these stealer panels through internet research such as Shodan, Using the query `http.html:"stealer"` and I discovered some Stealer Panels . Out of the three panels, two directly asked for login credentials to proceed further, while the third one appeared to be a proper website. We will analyze each of them one by one.
Stealer panels are mainly used to store stolen data. Some stealers are sold and purchased in underground markets, and based on these transactions, buyers receive credentials to access the panels. Through these panels, the owners can view and manage the stolen data, which often includes login credentials, financial details, and other sensitive information. These panels serve as a key tool for cybercriminals to organize and exploit stolen data efficiently.


### Panel 1 - PoisonX Stealer

![poisonx](https://github.com/user-attachments/assets/afa82556-345c-4059-9a87-6e258ca6b307)

`icon_hash=-145938113` 

Here are the following details where the infrastructure has been hosted.

| **ASN** | **IP Address** | **Hosted Countries** |
| --- | --- | --- |
| 51852 | `179.43.171.201` | Switzerland |
| 60077 | `193.151.136.249` | Iran |



## **Panel 2 -** Amatera Stealer

![amatera](https://github.com/user-attachments/assets/81ba59a2-fef0-4923-aab4-fa3e6092ae58)

`icon_hash="-2070047203”`

Here are the following details where the infrastructure has been hosted.

| **ASN** | **IP Address** | **Hosted Countries** |
| --- | --- | --- |
| 44066 | `84.200.154.182` | Germany  |
| 35913 | `45.89.196.115` | USA |



## **Panel 3 - L**ethalvoid

> icon_hash: 580969974
> 

ASN: 207279

**IP Address:** 194.48.248.64

**Hosted Country: USA**

Lethalvoid is a website that sells **ransomware** and provides various features to its users. It offers a **fully functional ransomware-as-a-service (RaaS) model**, allowing cybercriminals to purchase and customize ransomware for their attacks.

![lv_1](https://github.com/user-attachments/assets/137f1b93-8605-4786-a33c-6c687f0903ef)

Here is the first page of the Lethalvoid website, where you can see its description and purpose. The website openly promotes ransomware services, offering attackers the tools to encrypt victims' data .

![doc_1](https://github.com/user-attachments/assets/eb05f6fe-c880-4438-ba7e-0fac16fd24a4)

![doc_2](https://github.com/user-attachments/assets/7419747f-d8bd-4c1a-bbbc-0f3c2f024966)

This section contains the detailed documentation provided for users. The documentation likely serves as a guide for both free and paid users, helping them navigate the service efficiently.

![lv_2](https://github.com/user-attachments/assets/07aaf854-1e05-4278-a522-58245ea36c1c)

The service is divided into two categories: free users and paid users. Free users have limited access to basic functionalities, while paid users, who subscribe for $300 per year, get full access to advanced ransomware features.

![lv_3](https://github.com/user-attachments/assets/a9aad493-3dc7-46be-9218-769e42d371ba)

Here are the video evidences demonstrating the services provided by Lethalvoid. These videos showcase how the platform operates, including its features, functionalities, and the different tools available for users.

## Conclusion

Thank you everyone for reading my blog! This was my first attempt at performing network analysis on malicious IPs and stealer panels, and I learned a lot throughout the process. Since I am still exploring this field, there might be places where I could be wrong. Please feel free to share your insights or corrections.



