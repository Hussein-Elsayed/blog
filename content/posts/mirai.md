---
title: "Mirai HTB Writeup"
date: 2021-04-30T00:52:39+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "linux",
]
---
![Machine Info](/images/mirai/1.png)

* SYNOPSIS:
Mirai demonstrates one of the fastest-growing attack vectors in modern times; improperly
configured IoT devices. This attack vector is constantly rising as more and more IOT devices
are being created and deployed around the globe, and is actively being exploited by a wide
variety of botnets. Internal IoT devices are also being used for long-term persistence by malicious
actors.

### 1. Enumeration:
* Nmap:
![](/images/mirai/2.png)

Nmap reveals several open services: OpenSSH, a DNS server, a lighttpd server, and a Plex media
server with accompanying UPnP servers.
