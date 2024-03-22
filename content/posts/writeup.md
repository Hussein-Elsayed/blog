---
title: "Writeup HTB Walkthrough"
date: 2021-08-02T19:03:08+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "linux",
]
---
![Machine Info](/images/writeup/1.png)

### 1. Enumeration:
* Nmap:
To scan for open ports and services running
 * ``$ nmap -sV -sT -sC -o nmapscan 10.10.10.138``
![](/images/writeup/2.png)

![](/images/writeup/3.png)
![](/images/writeup/4.png)

adding the ip to our /etc/hosts file:
![](/images/writeup/5.png)

