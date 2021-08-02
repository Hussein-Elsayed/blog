---
title: "Traverxec HTB Writeup"
date: 2021-08-02T19:04:01+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "linux",
]
---
![Machine Info](/images/traverxec/1.png)

### 1. Enumeration:
* Nmap:
To scan for open ports and services running
 * ``$ nmap -sC -sV -A 10.10.10.165 -Pn``
![](/images/traverxec/2.png)

### 2. Web Enumeration:

![](/images/traverxec/3.png)
![](/images/traverxec/4.png)
![](/images/traverxec/5.png)
![](/images/traverxec/6.png)


### Dirsearch:

``$ python3 ~/dirsearch/dirsearch.py -u http://10.10.10.165/ -w /usr/share/wordlists/dirb/common.txt -e txt,html,php,log,zip,bac,bak,tar``
![](/images/traverxec/7.png)

nostromo 1.9.6:
![](/images/traverxec/8.png)

![](/images/traverxec/9.png)

nostromo web server and it seems to be vulnerable and has public exploits 
![](/images/traverxec/10.png)





