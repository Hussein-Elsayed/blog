---
title: "Nibbles HTB Writeup"
date: 2021-04-30T00:53:24+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "linux",
]
---
![Machine Info](/images/nibbles/1.png)

### 1. Enumeration:
* Nmap:
![](/images/nibbles/2.png)

### Web Enumeration:

* nothing useful!
![](/images/nibbles/3.png)

* let's check the source code:
 * found this comment containg a directory name..
![](/images/nibbles/4.png)
![](/images/nibbles/5.png)

Dirsearch:

``$ python3 ~/dirsearch/dirsearch.py -u http://10.10.10.75/nibbleblog/ -w /usr/share/wordlists/dirb/common.txt -e txt,html,php,log,zip,bac,bak,tar``
![](/images/nibbles/6.png)




