---
title: "CrossFit HTB Writeup"
date: 2021-03-31T15:35:44+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "insane",
    "linux",
]
---
![Machine info](/images/crossfit/1.png)

Hello guys, crossfit has retired today and here's its walkthrough. 

It's an amazing box; it contains lots of ideas. hope you enjoy it.

### 1. Enumeration: 
* Our Nmap scan as usual:
```yml
$ nmap -sC -sV 10.10.10.208

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ssl-cert: Subject: commonName=*.crossfit.htb/organizationName=Cross Fit Ltd./stateOrProvinceName=NY/countryName=US
| Not valid before: 2020-04-30T19:16:46
|_Not valid after:  3991-08-16T19:16:46
|_ssl-date: TLS randomness does not represent time
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 b0:e7:5f:5f:7e:5a:4f:e8:e4:cf:f1:98:01:cb:3f:52 (RSA)
|   256 67:88:2d:20:a5:c1:a7:71:50:2b:c8:07:a4:b2:60:e5 (ECDSA)
|_  256 62:ce:a3:15:93:c8:8c:b6:8e:23:1d:66:52:f4:4f:ef (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: Host: Cross; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
* From the scan: ``commonName=*.crossfit.htb/organizationName=Cross``

So let's add this host name to our /etc/hosts file 

![](/images/crossfit/2.png)

* let's see the web server:
just a regular apache server default page:
![](/images/crossfit/3.png)

* tried to connect to ftp >> there is no anonymous login available
![](/images/crossfit/4.png)

* let's see if there are hidden directories:
```text
$ python3 /home/sehs/dirsearch/dirsearch.py -u http://crossfit.htb -w /usr/share/dirb/wordlists/big.txt -e txt,html,php,log,zip,bac,bak,tar
```
![Found nothing](/images/crossfit/5.png)

### let's enumerate that FTP (port 21):
```text
$ nmap -p 21 --script=ftp* -oN ftpnmap.txt crossfit.htb
```
![](/images/crossfit/6.png)

further enumeration on port 21:
```text
$ nmap -sC -sV -p 21 -vvv 10.10.10.208
```
![](/images/crossfit/7.png)
Found this ``info@gym-club.crossfit.htb`` >> so there's a host named ``gym-club.crossfit.htb``
* let's add it in our /etc/hosts file and see:
![](/images/crossfit/8.png)

* Opening it up:
![](/images/crossfit/9.png)
![](/images/crossfit/10.png)
![](/images/crossfit/11.png)

some page not ready yet:
![](/images/crossfit/12.png)

http://gym-club.crossfit.htb/blog-single.php >> it has a comment form:
![](/images/crossfit/13.png)



















