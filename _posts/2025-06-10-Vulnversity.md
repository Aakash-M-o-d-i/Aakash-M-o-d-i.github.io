---
title: 'Tryhackme: Vulnversity'
date:  2025-06-10 23:00:00 +0530 
categories: [Tryhackme]
tags: [hacking, tryhackme, writeup, ctf, nmap, gobuster, reverse-shell, privilege-escalation, suid, systemctl, burpsuite, linux, walkthrough, flags, webshell, enumeration, exploitation, overpass, vulnversity] 
render_with_liquid: false
media_subpath: /images/Vulnversity/
---

# Vulnversity Walkthrough - 10 June 2k25

<div align="center">
    <img src="Vulnversity.png" alt="Vulnversity" width="400"/>
</div>

---

# Reconnaissance - Task 1

## Nmap Scan

**Command:**
```bash
sudo nmap -T4 -n -sC -sV -Pn -p- -oN fastscan.txt 10.10.224.12
```

![Nmap Scan Screenshot](nmap_scan_screenshot.png)

### Questions and Answers

| # | Question | Answer |
|---|----------|--------|
| 1 | Scan the box; how many ports are open? | `6` |
| 2 | What version of the squid proxy is running on the machine? | `3.5.12` |
| 3 | How many ports will Nmap scan if the flag -p-400 was used? | `400` |
| 4 | What is the most likely operating system this machine is running? | `ubuntu` |
| 5 | What port is the web server running on? | `3333` |
| 6 | What is the flag for enabling verbose mode using Nmap? | `-v` |

---

# Locating Directories using Gobuster - Task 2

## Directory Scan

**Command:**
```bash
gobuster dir -u http://10.10.187.173:3333/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt >> directory_scan.txt
```

**Question: 7** What is the directory that has an upload form page?  
**Answer:** `/internal/`

---

# Compromise the Webserver - Task 3

## Upload Directory and Reverse Shell

### Identifying Allowed Extensions

Use a wordlist to check which extensions are not blocked:
- `.php`
- `.php3`
- `.php4`
- `.php5`
- `.phtml`

![Wordlist](extension_check_wordlist.png)

---

### BurpSuite Configuration

BurpSuite is configured to intercept all browser traffic.

---

### Intruder

![Intruder](Intruder.png)

---

### Testing Extensions

![Test Extension](test_extension.png)

---

## Initialize the Reverse Shell

1. **Create the PHP Reverse Shell**  
   ![Reverse Shell](Reverse_shell.png)

2. **Listen on the Port**  
   ![Listen](listen.png)

3. **Upload the Exploit**  
   ![Upload Exploit](upload_exploit.png)

4. **Execute the Exploit**  
   ![Execute Exploit](execute_exploit.png)

---

### Questions and Answers

| # | Question | Answer |
|---|----------|--------|
| 8 | What common file type you'd want to upload to exploit the server is blocked? | `.php` |
| 9 | What extension is allowed after running the above exercise? | `.phtml` |
| 10 | What is the name of the user who manages the webserver? | `bill` |
| 11 | What is the user flag? | `8bd7992fbe8a6ad22a63361004cfcedb` |

**User Flag Screenshot:**  
![flag](flag.png)

---

# Privilege Escalation - Task 5

### SUID Enumeration

**Command:**
```bash
find / -perm -u=s -exec ls -l {} \; 2>/dev/null
```
![command suid](suid.png)

**Question: 12** On the system, search for all SUID files. Which file stands out?  
**Answer:** `/bin/systemctl`

---

## Gaining Root Access

1. **Create exploit service file**
    ```bash
    touch root.service
    ```
    ![exploit](exploit.png)

2. **Start a server to host the exploit**
    ```bash
    python3 -m http.server 8087
    ```
    ![server](server.png)

3. **Download the exploit on the victim machine**  
   ![download](download.png)

4. **Set up a listener and trigger the exploit**
    - Start listener:  
      ```bash
      nc -nlvp 4444
      ```
    - Enable and start the malicious service:
      ```bash
      systemctl enable /tmp/root.service
      systemctl start root
      ```
    ![successfull](successfull.png)

    **Root shell acquired!**

    
    ![done](done.png)

---

## Root Flag

![flag captured](flagc.png)

**Question: 13** What is the root flag value?  
**Answer:** `a58ff8579f0a9270368d33a9966c7fd5`

---

---

## Finish Note

**Congratulations on completing the Vulnversity walkthrough! This exercise demonstrates essential techniques for reconnaissance, exploitation, and privilege escalation. Always remember to practice ethical hacking and use these skills responsibly.**

Happy hacking!

## Happy Hacking

<div align="center">
    <img src="https://user-images.githubusercontent.com/74038190/212284119-fbfd994d-8c2a-4a07-a75f-84e513833c1c.gif" alt="Live Demo" width="400"/>
</div>
