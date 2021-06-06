---
title: "Cereal HTB Writeup"
date: 2021-05-29T08:27:20+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "hard",
    "windows",
]
---
![Machine Info](/images/cereal/1.png)

### 1. Enumeration:
* Nmap:
To scan for open ports and services
 * ``$ nmap -sC -sV 10.10.10.217``
![](/images/cereal/2.png)

So Three ports are opened 22:ssh  80:http  443:https
                        
There are two domain names in nmap result(DNS:cereal.htb, DNS:source.cereal.htb), Let's add them in our /etc/hosts file:
![](/images/cereal/3.png)

### 2. Web Enumeration:
* cereal.htb 
  * There is a simple login page:
![](/images/cereal/4.png)

* source.cereal.htb
  * gives us an error page contains path of file(C:\inetpub\source\default.aspx):
![](/images/cereal/5.png)

* Gobuster or dirsearch:

``$ gobuster dir -u http://source.cereal.htb/ -w /usr/share/  raft-small-words.txt -t 50`` or with common.txt wordlist
![](/images/cereal/6.png)

``$ python3 /home/sehs/dirsearch/dirsearch.py -u http://source.cereal.htb -w /usr/share/dirb/wordlists/raft-small-words.txt -e txt,html,php,log,zip,bac,bak,tar``
![](/images/cereal/7.png)

let's checkout the ``.git``
![](/images/cereal/8.png)
![](/images/cereal/9.png)

* it seems we could dump the .git repo to our localhost
  * using [GitTools](https://github.com/internetwache/GitTools) or [gitdumper](https://github.com/arthaud/git-dumper) 

```bash
git clone https://github.com/internetwache/GitTools
cd GitTools/Dumper/
ls
cat README.md
bash gitdumper.sh http://source.cereal.htb/.git/ dist-dir            
```
![](/images/cereal/10.png)

``$ bash gitdumper.sh http://source.cereal.htb/.git/  /home/sehs/Downloads/cereal/dump/``
![](/images/cereal/11.png)
![](/images/cereal/12.png)

* Extractor script:
  * extract commits and their content from a broken repository.

```text
cd ../Extractor/
ls
cat README.md
bash extractor.sh ../../dump/ dist-dir/all_dump/
```
![](/images/cereal/13.png)

``$ bash extractor.sh /home/sehs/Downloads/cereal/dump/ /home/sehs/Downloads/cereal/all_dump/``
![](/images/cereal/14.png)

So now we got our git files
![](/images/cereal/15.png)

And extracted them into all_dump
![](/images/cereal/16.png)

So it seems we got the source code of the application running ...
let's look for the authentication mechanism or any other interesting stuff ... 

* After roaming a bit, found secret:

![](/images/cereal/17.png)
![](/images/cereal/18.png)
![](/images/cereal/19.png)

var key = Encoding.ASCII.GetBytes("secretlhfIH&FY*#oysuflkhskjfhefesf");

``Secret : secretlhfIH&FY*#oysuflkhskjfhefesf``






































