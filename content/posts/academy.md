---
title: "Academy HTB Writeup"
date: 2021-05-01T04:27:56+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "linux",
]
---
![Machine info](/images/academy/1.png)

### 1. Enumeration:
* Nmap scan:
![](/images/academy/2.png)

let's add academy.htb to our hostnames:
![](/images/academy/4.png)

Wfuzz:
``$ wfuzz -u http://academy.htb/FUZZ.php -w /usr/share/dirb/wordlists/common.txt --hc 404,403 -t 100``
![](/images/academy/3.png)
