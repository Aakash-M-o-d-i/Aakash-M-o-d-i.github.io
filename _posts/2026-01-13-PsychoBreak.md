---
layout: post
title: "TryHackMe: PsychoBreak"
date: 2026-01-13 17:00:00 +0530
categories: [TryHackMe]
tags: [tryhackme, PsychoBreak, ctf, walkthrough, linux, web-enumeration, steganography, cryptography, ftp, ssh, hacking, writeup]
render_with_liquid: false
media_subpath: /images/PsychoBreak/
description: "Welcome to Beacon Mental Hospital. Help Sebastian and his team withstand the dangers that lie ahead in this Evil Within-themed CTF."

---

# TryHackMe: PsychoBreak CTF — Writeup | 13 January 2026

<div align="center">
    <img src="THM.png" alt="TryHackMe Logo" width="900"/>
    <img src="PsychoBreak.png" alt="Room Banner" width="600"/>
</div>

---

## Overview

Welcome to Beacon Mental Hospital. Sebastian Castellanos and his partners, Joseph Oda and Juli Kidman received a call from dispatch that there was just an incident at the hospital. So they began their investigation. Unfortunately, the team got separated.

Your job is to stand beside the team and help them withstand the challenges which are coming ahead...

This room is based on the video game **"The Evil Within"**. It's an easy-level CTF that involves enumeration, web exploitation, steganography, cryptography, and privilege escalation.

| Difficulty | Easy |
|:------:|:--------:|
| Time   | ~45 min  |
| OS     | Ubuntu   |

---

## Task 1: Reconnaissance & Scanning

### Nmap

Perform a full port and service scan:

```bash
sudo nmap -Pn -T4 -n -sC -sV -p- -oN scan_nmap.txt <TARGET_IP>
```

**Scan Summary:**

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.5a
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 44:2f:fb:3b:f3:95:c3:c6:df:31:d6:e0:9e:99:92:42 (RSA)
|   256 92:24:36:91:7a:db:62:d2:b9:bb:43:eb:58:9b:50:14 (ECDSA)
|_  256 34:04:df:13:54:21:8d:37:7f:f8:0a:65:93:47:75:d0 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Welcome To Becon Mental Hospital
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

**Open Ports:**
- **21/tcp:** FTP (ProFTPD 1.3.5a)
- **22/tcp:** SSH (OpenSSH 7.2p2)
- **80/tcp:** HTTP (Apache 2.4.18)

> **Answer:** How many ports are open? `3`  
> **Answer:** What is the operating system? `Ubuntu`

---

## Task 2: Web Enumeration

### The Locker Room

Navigate to the web page on port 80.

<div align="center">
    <img src="home_page.png" alt="Home Page" width="800"/>
</div>

Inspect the source code of the page. You'll find a hidden link that leads to a key for accessing the locker room. Click quickly — there might be a timer!

> **Answer:** Key to the Locker Room: 532219a04ab7a02b56faafbec1a4c1ea

---

### The Map - Atbash Cipher

In the locker room, you'll find an encoded message:

```
Tizmg_nv_zxxvhh_gl_gsv_nzk_kovzhv
```

This is an **Atbash Cipher**! In an Atbash cipher, each letter is replaced by its reverse in the alphabet (A↔Z, B↔Y, etc.).

Use an online decoder like [CyberChef](https://gchq.github.io/CyberChef/) or [dcode.fr](https://www.dcode.fr/atbash-cipher).

<div align="center">
    <img src="atbash_cipher.png" alt="Atbash Cipher" width="800"/>
</div>

> **Answer:** The Map Key: Grant_me_access_to_the_map_please

---

### Safe Haven - Directory Bruteforcing

The Safe Haven page has a hint in its source code to "search the page." Use **FUZZING** to find hidden directories:

```bash
ffuf -u http://<TARGET_IP>/SafeHaven/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<div align="center">
    <img src="ffuf.png" alt="FFUF Scan" width="800"/>
</div>

---

### The Keeper Key - Reverse Image Search

You'll encounter an image that holds the key. Perform a **reverse image search** using Google Images to identify the location shown in the picture.

<div align="center">
    <img src="keeper.png" alt="Keeper Key" width="800"/>
</div>

> **Answer:** The Keeper Key: 48ee41458eb0b43bf82b986cecf3af01
<div align="center">
    <img src="keeper_key.png" alt="Keeper Key" width="800"/>
</div>
---

## Task 3: Help Mee

In here we have to find the hidden files and extract them.
So for that their was a hint in abandonedRoom/URL page that we have to `shell` word to execute the linux commands.

So we have to use `ls+..` commands to get the hidden files.

<div align="center">
    <img src="ls_hidden.png" alt="LS Hidden" width="800"/>
</div>

### Analyzing helpme.zip

Download and extract `helpme.zip`:
<div align="center">
    <img src="hidden_url.png" alt="Hidden URL" width="800"/>
</div>

```bash
unzip helpme.zip
```

You'll find:
- `helpme.txt` - Contains a message about who is locked in the cell
- `Table.jpg` - A suspicious image file

> **Answer:** What is the filename of the text file (without extension)? `helpme`

---

### Corrupted Image - Hidden Zip

Check the true file type of `Table.jpg`:

```bash
file Table.jpg
```
<div align="center">
    <img src="file_table.jpg.png" alt="File Table.jpg" width="800"/>
</div>
It's actually a **ZIP file**! Rename and extract it:

```bash
mv Table.jpg Table.zip
unzip Table.zip
```

<div align="center">
    <img src="hidden_zip.png" alt="Hidden Zip" width="800"/>
</div>

---

### Morse Code Audio

Inside, you'll find a `.wav` audio file containing **Morse Code**. Use an online Morse code decoder:
- [morsecode.world](https://morsecode.world/international/decoder/audio-decoder-adaptive.html)

<div align="center">
    <img src="morse_code.png" alt="Morse Code" width="800"/>
</div>

---

### Steganography - FTP Credentials

The decoded Morse code gives you a passphrase. Use it with **steghide** to extract hidden data from `Joseph_Oda.jpg`:

```bash
steghide extract -sf Joseph_Oda.jpg
```

Enter the passphrase when prompted. This reveals the FTP credentials!

<div align="center">
    <img src="steghide.png" alt="Steghide" width="800"/>
</div>

<div align="center">
    <img src="thankyou_file.png" alt="Thank You File" width="800">
</div>

> **Answer:** FTP Username: joseph  
> **Answer:** FTP Password: intotheterror445
---

## Task 4: Crack It Open

### FTP Access

Connect to the FTP server using the discovered credentials:

```bash
ftp <TARGET_IP>
```

Download all files:

```bash
mget *
```

You'll find:
- `program` - An executable
- `random.dic` - A wordlist

<div align="center">
    <img src="ftp_access.png" alt="FTP Access" width="800"/>
</div>

---

### Brute-Forcing the Program

The `program` executable requires a key. Create a Python script to brute-force it:

```python
#!/usr/bin/env python3
import subprocess

wordlist = "random.dic"

with open(wordlist, "r", errors="ignore") as f:
    for word in f:
        word = word.strip()
        if not word:
            continue

        print(f"[*] Trying: {word}")

        result = subprocess.run(
            ["./program", word],
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            text=True
        )

        output = result.stdout + result.stderr

        # If output contains "Incorrect", continue
        if "Incorrect" in output:
            continue

        # Otherwise, success
        print("[+] Correct word found!")
        print(f"[+] Word: {word}")
        print("[+] Program output:")
        print(output)
        break

```

Make the program executable and run:

```bash
chmod +x program
python3 bruteforce.py
```

<div align="center">
    <img src="bruteforce.png" alt="Bruteforce" width="800"/>
</div>

> **Answer:** The key used by the program `kidman`

---

### T9/Multi-Tap Phone Cipher

The program outputs a string of numbers — this is a **T9 (Multi-tap) phone cipher**!

Use an online decoder like [dcode.fr](https://www.dcode.fr/multitap-abc-cipher) to decode it.

<div align="center">
    <img src="t9_cipher.png" alt="T9 Cipher" width="800"/>
</div>

> **Answer:** What do the crazy long numbers mean when decrypted? `KIDMANSPASSWORDISSOSTRANGE`

---

## Task 5: Go Capture The Flag

### SSH Access

Use the decoded credentials to SSH into the target:

```bash
ssh <kidman>@<TARGET_IP>
```

<div align="center">
    <img src="ssh_access.png" alt="SSH Access" width="800"/>
</div>

---

### User Flag

Navigate to the home directory and find the user flag:

```bash
cat user.txt
```

<div align="center">
    <img src="user_flag.png" alt="User Flag" width="800"/>
</div>

> **Answer:** What is the user flag? `4C72A4EF8E6FED69C72B4D58431C4254`

---

## Root Privilege Escalation
command: `find / -perm -4000 -type f 2>/dev/null`

<div align="center">
    <img src="find_4000.png" alt="Find 4000" width="800">
</div>

We get `/usr/bin/pkexec` with SUID bit set.
so `pkexec` is a vulnerable binary.
Let's try to exploit it. Here is the github link for the exploit: [pkexec](https://github.com/ly4k/PwnKit?tab=readme-ov-file)

<img src="pkexec_exploit.png" alt="pkexec" width="800">

### Root Flag

Elevate privileges and capture the root flag:

```bash
cat /root/root.txt
```

<div align="center">
    <img src="root_flag.png" alt="Root Flag" width="800"/>
</div>

> **Answer:** What is the root flag? `BA33BDF5B8A3BFC431322F7D13F3361E`

---

## Bonus: Defeat Ruvik

Check for other users on the system:

```bash
ls /home/
cat /etc/passwd | grep ruvik
```

Look for a cron job or script running as root:

```bash
cat /etc/crontab
```

You might find `.the_eye_of_ruvik.py` being executed with root privileges. Exploit this for privilege escalation!

Just run this command: sudo deluser --remove-home ruvik
This will remove the ruvik user and delete its home directory.

<div align="center">
    <img src="defeat_ruvik.png" alt="Defeat Ruvik" width="800"/>
</div>

> **Answer:** [Bonus] Defeat Ruvik: mark as complete

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
| Web | Source code analysis | Browser DevTools |
| Web | Atbash cipher | CyberChef/dcode.fr |
| Web | Directory bruteforcing | Gobuster |
| Web | Reverse image search | Google Images |
| Help Mee | File type analysis | `file` command |
| Help Mee | Morse code decoding | Audio decoder |
| Help Mee | Steganography | Steghide |
| Crack It | FTP enumeration | FTP client |
| Crack It | Program brute-force | Python scripting |
| Crack It | T9/Multi-tap cipher | dcode.fr |
| Capture Flag | SSH access | SSH client |
| Bonus | Privilege escalation | Cron job exploitation |

---

## Happy Hacking!

<div align="center">
    <img src="https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExbnNrb2VzdnJsZXNvNTY0bzk2cWFoNXY4b241OXYza2c3c21uaHh1MiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/EJGsKGyu9r48KgiOB4/giphy.gif" alt="Hacking GIF" width="800"/>
</div>
