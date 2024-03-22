---
title: "Admirer HTB Writeup"
date: 2021-05-01T04:28:49+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "linux",
]
---
![Machine info](/images/admirer/1.png)

### 1. Enumeration:

* Nmap scan:
![](/images/admirer/2.png)

Web Enumeration:

![](/images/admirer/3.png)

checking robots.txt >> found this ``/admin-dir``
![](/images/admirer/4.png)

visiting ``/admin-dir/credentials.txt``
```text
[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
```
![](/images/admirer/5.png)

Gobuster:

``$gobuster dir -w /usr/share/golismero/wordlist/fuzzdb/Discovery/PredictableRes/raft-medium-directories.txt -u 10.10.10.187/admin-dir/ -x php,txt -s 200``
![](/images/admirer/6.png)
![](/images/admirer/7.png)

revealed this ``/contacts.txt``
```text
##########
# admins #
##########
# Penny
Email: p.wise@admirer.htb

##############
# developers #
##############
# Rajesh
Email: r.nayyar@admirer.htb

# Amy
Email: a.bialik@admirer.htb

# Leonard
Email: l.galecki@admirer.htb

#############
# designers #
#############
# Howard
Email: h.helberg@admirer.htb

# Bernadette
Email: b.rauch@admirer.htb
```
![](/images/admirer/8.png)

FTP enumeration:
logging in with ftpuser and password: ``%n?4Wz}R$tTF7``
![](/images/admirer/9.png)

login succeeded , let's dump these files:

``$ wget --user ftpuser --password '%n?4Wz}R$tTF7' -m ftp://10.10.10.187``
![](/images/admirer/10.png)

So we downloaded the files ``dump.sql`` and ``html.tar.gz``

* dump.sql >> has nothing useful

![](/images/admirer/11.png)
![](/images/admirer/12.png)

* index.php:

```
$servername = "localhost";
                        $username = "waldo";
                        $password = "]F7jLHw:*G>UPrTo}~A"d6b";
                        $dbname = "admirerdb";
```
![](/images/admirer/13.png)

* looking in ``w4ld0s_s3cr3t_d1r``:

```text
[Bank Account]
waldo.11
Ezy]m27}OREc$

[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
```
![](/images/admirer/14.png)

looking in ``/utility-scripts/db_admin.php``:

```text
$servername = "localhost";
  $username = "waldo";
  $password = "Wh3r3_1s_w4ld0?";
```
![](/images/admirer/15.png)

checking ``/utility-scripts``:
![](/images/admirer/16.png)

Gobuster again:

``$ gobuster dir -u http://10.10.10.187/utility-scripts/ -w /usr/share/dirb/wordlists/big.txt -t 30 -x php,txt -s 200``
![](/images/admirer/17.png)

found this ``/adminer``
![](/images/admirer/18.png)

![](/images/admirer/19.png)



