---
title: "Bastion HTB Writeup"
date: 2021-08-02T19:02:58+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "windows",
]
---
![Machine Info](/images/bastion/1.png)

### 1. Enumeration:
* Nmap:
To scan for open ports and services running
 * ``$ nmap -sC -sV -A 10.10.10.134 -Pn``
![](/images/bastion/2.png)

so we got 
```
ssh           on                22 
RPC           on                135
netbios-ssn   on                139
microsoft-ds  on                445
```
about these [ports](https://www.youtube.com/watch?v=KOyZdnjUSQg&t=12s)

![](/images/bastion/3.png)
![](/images/bastion/4.png)
![](/images/bastion/5.png)

adding it to our /etc/hosts file
![](/images/bastion/6.png)




