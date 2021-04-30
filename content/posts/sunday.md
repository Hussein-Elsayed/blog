---
title: "Sunday HTB Writeup"
date: 2021-04-30T00:54:03+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
]
---
![Machine Info](/images/sunday/1.png)

### 1. Enumeration:
* Nmap:
![](/images/sunday/2.png)

another scan for all the TCP ports:
* ``$ nmap -sC -sV -p- -T4 --min-parallelism=50 -n --min-rate=300 -o nmapscan 10.10.10.76 ``
![](/images/sunday/3.png)

Best result: 
scanning all ports but took too much time
``$ nmap -p 1-65535 -T4 -A 10.10.10.76``
![](/images/sunday/4.png)

Anyway we have these interesting ports and services:
* 79		finger 
* 111, 52988	rpcbind
* 22022		ssh

Let's do some search:

``A finger service is running on this host. The finger protocol is used to find out information about users on a remote system. Finger servers can usually provide either a list of logged-in users or detailed information on a single user.``

we can enumerate finger with this [finger-user-enum](https://github.com/pentestmonkey/finger-user-enum) tool..
* Usage:
![](/images/sunday/5.png)

``$ ./finger-user-enum.pl -U /usr/share/SecLists/Usernames/Names/names.txt -t 10.10.10.76``
![](/images/sunday/6.png)

we got sammy and sunny users and root..also their connection type!

Notice the difference between (tty and pts):

![](/images/sunday/7.png)

which means there is an ssh connection from user sunny and ofcourse will be on port 22022 not the default 22..

Getting User:

* Trying sunny and password: sunday as a guess >> worked
* didn't find the user.txt flag..
![](/images/sunday/8.png)

Trying ``sudo -l`` >> works fine without password:

* We can run the /root/troll script with root priv without password.
![](/images/sunday/9.png)
![](/images/sunday/10.png)

Found sammy hash:
``$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445``
![](/images/sunday/11.png)

let's crack it with John:
* save it to file >> hash.txt
![](/images/sunday/12.png)

* ``$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt``
* ``$ john --show hash.txt``
![](/images/sunday/13.png)

GOT the password for sammy: ``cooldude!``

Switching into sammy >> didn't get the user flag!
![](/images/sunday/14.png)

logging with ssh is better:
* you will get an error >> and to solve it use this flag 
``-oKexAlgorithms=+diffie-hellman-group1-sha1``
* ``ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 sammy@10.10.10.76 -p 22022``
* All good and got the user flag..
![](/images/sunday/15.png)

### Privilege Escalation:

* ``sudo -l`` command needs no password and reveals that we could use wget with root privileges:
![](/images/sunday/16.png)

There are lots of ways to privilege with wget if it runs with root privilege!








