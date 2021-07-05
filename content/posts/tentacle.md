---
title: "Tentacle HTB Writeup"
date: 2021-06-20T11:59:31+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "hard",
    "linux",
]
---
![Machine Info](/images/tentacle/1.png)

### 1. Enumeration:
* Nmap:
To scan for open ports and services running
 * ``$ nmap -sC -sV -A -T4 10.10.10.224 -Pn``
![](/images/tentacle/2.png)
