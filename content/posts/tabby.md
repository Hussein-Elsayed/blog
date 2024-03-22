---
title: "Tabby HTB Writeup"
date: 2021-05-01T04:28:33+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "linux",
]
---
![Machine info](/images/tabby/1.png)

### 1. Enumeration:
* Nmap scan:
![](/images/tabby/2.png)

### Web Enumeration:

* PORT 80:
![](/images/tabby/3.png)

* PORT 8080:
![](/images/tabby/4.png)

tomcat is running and this is its initial page
![](/images/tabby/5.png)

getting to port 80:

The url looks very suspicious because it including the statement file in a lfi way
![](/images/tabby/6.png)

local file inclusion:
 * ``view-source:http://10.10.10.194/news.php?file=../../../../../../../../../etc/passwd``
![](/images/tabby/7.png)

As I found LFI vulnerability then my next step is to find a way by which we can perform Remote Code execution on target machine through which we can open a shell on our PC to access the machine.




