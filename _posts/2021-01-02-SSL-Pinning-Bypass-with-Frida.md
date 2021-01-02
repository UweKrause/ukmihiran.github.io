---
title: SSL Pinning Bypass with Frida for Beginners
author: Udesh
date: 2021-01-02
categories: [Pentesting]
tags: [Pentesting]
image: /assets/img/post/post8/header.png
---

This article will explain how to bypass SSL pinning of android applications using Frida framework. So basically Frida is a tool that let you inject scripts to native apps (in this case Android apps) to modify the application behavior (in this case, SSL pinning bypass and can perform a MitM attack, even if the application has https / SSL connections) and make dynamic test in real time.

In this blog post will include:
- How to configure & install Burp certificate for android device.
- How to connect android device with adb.
- Install Frida server. 
- How to invoke the script with your application.

Pre-Conditions:
- You'd need an android virtual machine. (NoxPlayer/Genymotion)

## Step 1: Install Burp Certificate

First, we need to configure our burp suite proxy.

![1.PNG]({{site.baseurl}}/assets/img/post/post8/1.PNG)

We have configured our proxy as bind address 192.168.8.127 and bind port 8080. Now, go to mobile phone browser and navigate the proxy's IP and port (http://192.168.8.127:8080) and download the proxy's certificate by clicking on **CA certificate**.

![2.PNG]({{site.baseurl}}/assets/img/post/post8/2.PNG)

Once you downloaded the CA certificate, we need to rename it **cacert.der** to **burp.cer**

![3.PNG]({{site.baseurl}}/assets/img/post/post8/3.PNG)

After that in the android device go to **settings -> security ->**  under credential storage select **Install from SD card**, navigate to where the **burp.cer** certificate is located and select it, enter a name whatever you like and you will most likely be asked to set a Lock Screen PIN or a password, do it and you will see a *burp installed* message.

![4.PNG]({{site.baseurl}}/assets/img/post/post8/4.PNG)

Now we can configure proxy settings in our virtual mobile device. Go to the virtual device **Settings > Wi-fi > WiredSSID** and long press on it. It will popup WiredSSID window and you need to modify Network, on the proxy dropdown select **Manual** and you will be able to enter which proxy to connect. In this case I entered Proxy hostname -192.168.8.127 and Proxy port – 8080.

![5.PNG]({{site.baseurl}}/assets/img/post/post8/5.PNG)

## Step 2: Connect to Android Device with adb

To run the commands on android device we need to connect our device to adb. But to do that we need to go to the settings of the device or the emulator then to **Developer Options** and start the **USB-debugging mode**.

After that we can run the following command to connect our android device.
```shell
adb connect <IP address of the android device>
```
You can check if the device is connected to adb.
```shell
adb devices 
```
![6.PNG]({{site.baseurl}}/assets/img/post/post8/6.PNG)

## Step 3: Install Frida Server

First, you need to install Frida Tools on your machine. you can install Frida-tools via terminal.
```shell
pip install frida-tools
```
After that you can just run the `Frida --help` to make sure it was correctly installed. 

![7.PNG]({{site.baseurl}}/assets/img/post/post8/7.PNG)

We have installed Frida-tools properly. Now, we need to download Frida server executable file. Before download the Frida server we need to find out what type of processor architecture have our device. you can enter following command to find it.

```shell
adb shell getprop ro.product.cpu.abi
```
output: x86

![8.PNG]({{site.baseurl}}/assets/img/post/post8/8.PNG)

Well, after knowing the arch then you can download the properly Frida server version for our device. download it from this [link](https://github.com/frida/frida/releases "link"). In this case I have downloaded frida-server-14.1.0-android-x86 version of Frida server. 

Once you downloaded it, we need to copy the Frida server binary to the android device with adb.

```shell
adb push ./ frida-server-14.1.0-android-x86 /data/local/tmp/Frida-server
```
Give execution permission to Frida server:
```shell
adb shell chmod 755 /data/local/tmp/frida-server
```
Now. We can run the Frida server with following command.
```shell
adb shell /data/local/tmp/frida-server &
```
The last **'&'** is to run the command in background.

Now, check if Frida is working correctly, for this, run following command.
```shell
frida-ps -U
```
![9.PNG]({{site.baseurl}}/assets/img/post/post8/9.PNG)

## Step 4: Bypass SSL Pinning 
Now, we need to get the frida script that will let you override SSL connections to create and use our own Trust Manager. You can find the script [here](https://codeshare.frida.re/@akabe1/frida-multiple-unpinning/ "here")

We will first see to use this script to bypass SSL Pinning and then we will analyze what the script does. With the script at hand, you can run the next command.

In this scenario our target is twitter. (com.twitter.android)

```shell
frida -U -f com.twitter.android -l sslbypass.js --no-pause
```
![10.PNG]({{site.baseurl}}/assets/img/post/post8/10.PNG)

Once all things go well, all traffic of the target app will get intercepted into Burp Suite. 😎

![11.PNG]({{site.baseurl}}/assets/img/post/post8/11.PNG)
