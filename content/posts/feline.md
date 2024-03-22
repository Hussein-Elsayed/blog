---
title: "Feline HTB Writeup"
date: 2021-05-01T04:30:17+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "hard",
    "linux",
]
---
![Machine info](/images/feline/1.png)

### 1. Enumeration:
* Nmap scan:
```text
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
8080/tcp open  http    Apache Tomcat 9.0.27
|_http-title: VirusBucket
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
![](/images/feline/2.png)

let's add this ``VirusBucket`` to our hostnames:
![](/images/feline/3.png)

### Web Enumeration:

* PORT 8080:
![](/images/feline/4.png)

![](/images/feline/5.png)







