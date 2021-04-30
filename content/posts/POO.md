---
title: "P.O.O Endgame HTB Writeup"
date: 2021-04-30T00:55:54+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "endgame",
    "active directory",
]
---
![Machines Info](/images/poo/1.png)

As you see endgame type consists of more than one machine connected to each other and the flags are devided on specific steps..

### 1. Enumeration:
* Nmap:
 * ``$ nmap -sV -sC -A 10.13.38.11 -Pn``
![](/images/poo/2.png)

### Web Enumeration:
* PORT 80 
 * iis default page..
![](/images/poo/3.png)

Nikto:
* simple web vuln scanner
``$ nikto -h 10.13.38.11``
![](/images/poo/4.png)

nikto revealed a ``.DS_Store`` file in the server's root folder. 

```text
The DS_Store, or
Desktop Services Store is a hidden file use by Mac OS X. This file is used to store various 
attributes about the folder such as icons or sub-folder names. This file can reveal sensitive
information such as the folder structure and contained files.
```

* visiting ``http://10.13.38.11/.DS_Store``
![](/images/poo/5.png)

we can use this [DS_walk](https://github.com/Keramas/DS_Walk) tool to enumeate the files 
![](/images/poo/6.png)

Usage:
![](/images/poo/7.png)

* ``$ python ds_walk.py -u http://10.13.38.11/``
![](/images/poo/8.png)

The interesting ones > ``/admin , /dev`` and their sub-dirctories

They all give this inaccessible error
![](/images/poo/9.png)

admin gives login prompt and unauthorized access 
![](/images/poo/10.png)

Let's save these results and continue with enumeration..

IIS short name enumeration:

Searching about iis v 7.5 vulnerabilities and misconfigurations found this [paper](https://soroush.secproject.com/downloadable/microsoft_iis_tilde_character_vulnerability_feature.pdf) or this [article](https://support.detectify.com/support/solutions/articles/48001048944-microsoft-iis-tilde-vulnerability) about iis tilde character ``~`` 
which reveals files and folders names and also extentions
![](/images/poo/11.png)

we can use a metasploit module for this or this [github](https://github.com/lijiejie/IIS_shortname_Scanner) tool 

* use auxiliary/scanner/http/iis_shortname_scanner
* show options
* set rhosts 10.13.38.11
* run
![](/images/poo/12.png)
found this > but not interesting 

using the tool pretty the same:

``$ python iis_shortname_Scan.py http://10.13.38.11/``
![](/images/poo/13.png)

Anyway we need to repeat the same search on the sub-directories too:
![](/images/poo/14.png)

so let's set the /path parameter and see what we get 

set path /dev/304c0c90fbc6520610abbf378e2339d1
set path /dev/304c0c90fbc6520610abbf378e2339d1/db
![](/images/poo/15.png)

looks interesting ``/poo_co*~1.txt*`` >> got it from both the tool and metasploit.

As the shortname only contains 6 characters, the rest of the file name should be guessed or discovered manually. 

Let's extract all words starting with ``co`` and fuzz the filename with this wordlist..

``$ grep '^co.*' /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt > ./fuzz.txt``
![](/images/poo/16.png)

* Wfuzz:
 * ``$ wfuzz -z file,fuzz.txt -t 50 --sc 200 -u http://10.13.38.11/dev/304c0c90fbc6520610abbf378e2339d1/db/poo_FUZZ.txt``
![](/images/poo/17.png)

so it's connection >> /poo_connection.txt
* /dev/304c0c90fbc6520610abbf378e2339d1/db/poo_connection.txt
![](/images/poo/18.png)

got the first flag: 
POO{fcfb0767f5bd3cbc22f40ff5011ad555}

## Huh?!


