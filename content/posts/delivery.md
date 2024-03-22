---
title: "Delivery HTB Writeup"
date: 2021-05-24T00:36:53+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "linux",
]
---
![Machine Info](/images/delivery/1.png)

### 1. Enumeration:
* Nmap:
To scan for open ports and services
 * ``$ nmap -sC -sV 10.10.10.222``
![](/images/delivery/2.png)

so simply we got only ssh (port 22), web server (port 80)

### 2. Web Enumeration:

![](/images/delivery/3.png)

press ctrl+u >> to check page source code

![](/images/delivery/4.png)
![](/images/delivery/5.png)

so we got these hostnames: ``http://delivery.htb:8065/`` and ``http://helpdesk.delivery.htb/``

* let's add them to our ``/etc/hosts`` file.. 

* ``10.10.10.222	helpdesk.delivery.htb	delivery.htb``

![](/images/delivery/6.png)








