---
title: "Heist HTB Writeup"
date: 2021-08-02T19:03:27+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "windows",
]
---
![Machine Info](/images/heist/1.png)

### 1. Enumeration:
* Nmap:
To scan for open ports and services running
 * ``$ nmap -sC -sV -o nmapscan 10.10.10.149``
![](/images/heist/2.png)

let's add it to our /etc/hosts file 
![](/images/heist/3.png)

### 2. Web Enumeration:
![](/images/heist/4.png)

tried admin:password but it asks for email not username , tried admin@htb , gave this error page:
![](/images/heist/5.png)

let's try login as guest:
![](/images/heist/6.png)

so we got user hazard asking the admin a question and this interesting attachment (config file) > let's check it:
![](/images/heist/7.png)

```
security passwords min-length 12
enable secret 5 $1$pdQG$o8nrSzsGXeaduXrjlvKc91

username rout3r password 7 0242114B0E143F015F5D1E161713
username admin privilege 15 password 7 02375012182C1A1D751618034F36415408
```

another nmap scan shows two other open ports:
``$ nmap -sV -sT -p- -o fullportscan heist.htb``
![](/images/heist/8.png)
```
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49669/tcp open  msrpc         Microsoft Windows RPC
```
the port 5985 is for [winRM](https://book.hacktricks.xyz/pentesting/5985-5986-pentesting-winrm)
![](/images/heist/9.png)










