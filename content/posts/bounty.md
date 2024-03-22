---
title: "Bounty HTB Writeup"
date: 2021-04-30T00:54:09+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "windows",
]
---
![Machine Info](/images/bounty/1.png)

### 1. Enumeration:
* Nmap:
![](/images/bounty/2.png)

### Web Enumeration:
* visiting the website >> nothing useful 
![](/images/bounty/3.png)

wfuzz:
``$ wfuzz -c --hc 404 -t 200 -w /usr/share/wordlists/dirb/common.txt http://bounty.htb/FUZZ``

Gobuster:
since it's iis server , let's look for aspx extentions:
``$ gobuster dir -u http://bounty.htb -x aspx -w /usr/share/wordlists/dirb/common.txt -t 20``
![](/images/bounty/4.png)