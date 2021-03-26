---
title: "Time HTB Writeup"
date: 2021-03-26T4:42:47+02:00
draft: false
---
![The machine info](/images/1.png)

* **Enumeration:**
  * Nmap scan:
![](/images/2.png)

* let's checkout the running web server:
![](/images/3.png)

its a java beautifier and validator.

* lets check the source code:
![](/images/4.png)
nothing interesting in the source code

* let's search for hidden directories:
```yml
$ python3 dirsearch.py -u http://10.10.10.223/ -e txt,html,php,log,zip,bac,bak,tar
```
![](/images/5.png)
![](/images/6.png)