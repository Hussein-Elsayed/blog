---
title: "Forest HTB Writeup"
date: 2021-08-02T19:03:34+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "windows",
]
---
![Machine Info](/images/forest/1.png)

### 1. Enumeration:
* Nmap:
To scan for open ports and services running
 * ``$ nmap -sC -sV -A 10.10.10.161 -Pn``
![](/images/forest/2.png)

```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec?
135/tcp  open  msrpc?
139/tcp  open  netbios-ssn?
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds  Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
```

636 ldaps ,  389 ldap  , 88 kerberos , 3269 global catalog (LDAP in ActiveDirectory) , 464 something with a domain controller

593 rpc over http


