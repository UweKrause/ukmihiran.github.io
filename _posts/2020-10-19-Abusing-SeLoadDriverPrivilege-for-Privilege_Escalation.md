---
title: Abusing SeLoadDriverPrivilege for Privilege Escalation
author: Udesh
date: 2020-10-19
categories: [Windows, Privilege Escalation]
tags: [Privilege Escalation]

---
# Abusing Windows SeLoadDriverPrivilege for Privilege Escalation

Windows **SeLoadDriverPrivilege** gives service privilege to load and unload device drivers. this service allows user to install and remove drivers for Plug and Play devices. Assigning certain privileges to user accounts without administration permissions can result in local privilege escalation attacks. 

Lets analyze the impact associated to the assignment of the “**Load and unload device drivers**” policy.
#### Prepping
Checking if our user to which the privilege “**Load and unload device drivers**” has been assigned.
```shell
Whoami /all
```
The above command shows the user information, groups and privileges. In this scenario, **Load and unload device drivers** privilege enable by default.

![whoami.PNG]({{site.baseurl}}/assets/img/post/post3/whoami.PNG)


#### Elevate Privilege
For the privilege escalation, we have to compile 2 important files.

**Eoploaddriver.cpp**
[https://raw.githubusercontent.com/TarlogicSecurity/EoPLoadDriver/master/eoploaddriver.cpp](https://raw.githubusercontent.com/TarlogicSecurity/EoPLoadDriver/master/eoploaddriver.cpp)

**ExploitCapcom.cpp**
[https://github.com/tandasat/ExploitCapcom](https://github.com/tandasat/ExploitCapcom) =>This exploit allows you to obtain a Shell as SYSTEM

And load the malicious kernel driver  **Capcom.sys**

This great [article](https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/) talk about this vulnerability and its PoC

You can find of mine [Here](https://github.com/ukmihiran/HTB-Stuff/tree/master/HTB-FUSE-script "Here"). modify netcat.bat and put all of them to c:\temp, then make it run. here, I used revershell from remote machine to my machine.

```shell
.\EOPLOADDRIVER.exe System\CurrentControlSet\MyService C:\temp\capcom.sys
.\ExploitCapcom_modded.exe
```

![exploit.PNG]({{site.baseurl}}/assets/img/post/post3/exploit.PNG)


Make sure listening nc -lvnp. after that you will get connected from the remote machine to your shell. 😎
