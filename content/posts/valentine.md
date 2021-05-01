---
title: "Valentine HTB Writeup"
date: 2021-04-30T00:53:55+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "linux",
]
---
![Machine Info](/images/valentine/1.png)

### 1. Enumeration:
* Nmap:
![](/images/valentine/2.png)

### Web Enumeration:
* http(PORT 80)
![](/images/valentine/3.png)

* https(PORT 443) >> gives the same result(the same page and source code)

Let's add this hostname (valentine.htb) from nmap to our /etc/hosts file..
![](/images/valentine/4.png)
visiting the website >> same result nothing different..

Time for Gobuster:
* ``$ gobuster dir -u http://Valentine.htb/ -w /usr/share/wordlists/dirb/common.txt -t 30``
![](/images/valentine/5.png)



