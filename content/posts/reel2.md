---
title: "Reel2 HTB Writeup"
date: 2021-05-01T04:30:39+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "hard",
    "windows",
]
---
![Machine info](/images/reel2/1.png)

### 1. Enumeration:
* Nmap scan:
![](/images/reel2/2.png)

http(80), https(443), http-proxy(8080) are open

### Web Enumeration:

* PORT 80:
![](/images/reel2/3.png)

* PORT 443:
![](/images/reel2/4.png)

* PORT 8080:
![](/images/reel2/5.png)

Gobuster:

``$ gobuster dir -u https://reel2.htb/ -w /usr/share/dirb/wordlists/big.txt -b 404,403 -k``
![](/images/reel2/6.png)
![](/images/reel2/7.png)
![](/images/reel2/8.png)

```text 
/ews (Status: 301)
/exchange (Status: 302)
/exchweb (Status: 302)
/owa (Status: 301)
/public (Status: 302)
/rpc (Status: 401)
```

``/owa`` >> outlook web app and needs login creds:
![](/images/reel2/9.png)

lets get back to 8080 and signup
![](/images/reel2/10.png)
![](/images/reel2/11.png)

![](/images/reel2/12.png)
![kinda interesting](/images/reel2/13.png)

lets gather all the usernames here:
```text
admin1
sven
svensson
cube
egre55
cube0x0
lars
larsson
jeenny
adams
teresa
trump
wtf
admin
```
lets make a list of them:
![](/images/reel2/14.png)

lets use this python script on names.txt to make combinations of 'em:
```yml
#!/usr/bin/env python
import sys
import os.path

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("usage: {} names.txt".format((sys.argv[0])))
        sys.exit(0)

    if not os.path.exists(sys.argv[1]):
        print("{} not found".format(sys.argv[1]))
        sys.exit(0)

    for line in open(sys.argv[1]):
        name = ''.join([c for c in line if  c == " " or  c.isalpha()])

        tokens = name.lower().split()

        # skip empty lines
        if len(tokens) < 1:
            continue

        fname = tokens[0]
        lname = tokens[-1]

        print(fname + lname)           # johndoe
        print(lname + fname)           # doejohn
        print(fname + "." + lname)     # john.doe
        print(lname + "." + fname)     # doe.john
        print(lname + fname[0])        # doej
        print(fname[0] + lname)        # jdoe
        print(lname[0] + fname)        # djoe
        print(fname[0] + "." + lname)  # j.doe
        print(lname[0] + "." + fname)  # d.john
        print(fname)                   # john
        print(lname)                   # joe

```
![](/images/reel2/15.png)

./script.py names.txt >> result.txt
![](/images/reel2/16.png)

![](/images/reel2/17.png)