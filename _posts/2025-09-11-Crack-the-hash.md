---
layout: post
title: 'TryHackMe: Crack the hash'
date: 2025-09-12 23:00:00 +0530
categories: [TryHackMe]
tags: [hash-cracking, cryptography, john, hashcat, pentesting]
render_with_liquid: false
media_subpath: /images/Crack_the_hash/
---

# TryHackMe: Crack the hash — Writeup | 12 September 2025

<div align="center">
  <img src="THM.png" alt="TryHackMe Logo" width="800"/>
  <img src="Crack-the-hash.png" alt="Room Banner" width="800"/>
</div>

**Author:** Aakash Modi

## Overview

This room focuses on cracking different types of hashes using various tools like hashcat and John the Ripper. We'll learn about hash identification and cracking techniques.

## Tools Used
- hashcat
- John the Ripper
- hash-identifier
- online hash crackers

---

## Level 1 Challenges

### Hash 1: MD5
```
48bb6e862e54f2a795ffc4e541caed4d
```

**Analysis:**
- Length: 32 characters
- Type: MD5 (Message Digest 5)
- Tool Used: hashcat -m 0

<div align="center">
  <img src="level_1_hash.png" alt="Hash 1 Cracking" width="800"/>
</div>

**Cracked Value:** easy

### Hash 2: SHA1
```
CBFDAC6008F9CAB4083784CBD1874F76618D2A97
```

**Analysis:**
- Length: 40 characters
- Type: SHA1 (Secure Hash Algorithm 1)
- Tool Used: John the Ripper with rockyou.txt

<div align="center">
  <img src="level_1_hash2.png" alt="Hash 2 Cracking" width="800"/>
</div>

**Cracked Value:** password123

### Hash 3: SHA256
```
1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
```

**Analysis:**
- Length: 64 characters
- Type: SHA256
- Tool Used: hashcat -m 1400

<div align="center">
  <img src="level_1_hash3.png" alt="Hash 3 Cracking" width="800"/>
</div>

**Cracked Value:** letmein

### Hash 4: bcrypt
```
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
```

**Analysis:**
- Prefix: $2y$ indicates bcrypt
- Cost Factor: 12
- Tool Used: John the Ripper with custom rules

<div align="center">
  <img src="level_1_hash4.png" alt="Hash 4 Cracking" width="800"/>
</div>

**Cracked Value:** bleh

### Hash 5: MD5
```
279412f945939ba78ce0758d3fd83daa
```

**Analysis:**
- Length: 32 characters
- Type: MD5
- Tool Used: Online MD5 decoder

<div align="center">
  <img src="level_1_hash5.png" alt="Hash 5 Cracking" width="800"/>
</div>

**Cracked Value:** Eternity22

---

## Level 2 Challenges

### Hash 1: SHA256
```
F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
```

**Analysis:**
- Length: 64 characters
- Type: SHA256
- Tool Used: hashcat with custom wordlist

<div align="center">
  <img src="level_2_hash1.png" alt="Level 2 Hash 1" width="800"/>
</div>

**Cracked Value:** paule

### Hash 2: MD4
```
1DFECA0C002AE40B8619ECF94819CC1B
```

**Analysis:**
- Length: 32 characters
- Type: MD4
- Tool Used: John the Ripper

<div align="center">
  <img src="level_2_hash2.png" alt="Level 2 Hash 2" width="800"/>
</div>

**Cracked Value:** n63umy8lkf4i

### Hash 3: SHA512crypt
```
$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.
```

**Analysis:**
- Type: SHA512crypt with salt
- Prefix: $6$ indicates SHA512crypt
- Salt: "aReallyHardSalt"
- Tool Used: hashcat -m 1800

<div align="center">
  <img src="level_2_hash3_page.png" alt="Level 2 Hash 3 Page" width="800"/>
  <img src="level_2_hash3_found.png" alt="Level 2 Hash 3 Found" width="800"/>
</div>

**Cracked Value:** waka99

### Hash 4: SHA1
```
e5d8870e5bdd26602cab8dbe07a942c8669e56d6
```

**Analysis:**
- Length: 40 characters
- Type: SHA1
- Tool Used: Online SHA1 cracker

<div align="center">
  <img src="level_2_hash4.png" alt="Level 2 Hash 4" width="800"/>
</div>

**Cracked Value:** [Hash still in cracking process]

## Key Takeaways
1. Always identify the hash type before attempting to crack
2. Different tools excel at different hash types
3. Salt significantly increases cracking difficulty
4. Modern hash algorithms (bcrypt, SHA512crypt) are more resistant to cracking

---

## Room Complete!

<div align="center">
  <img src="completed.png" alt="Completed" width="800"/>
</div>

---

## Happy Hacking!

<div align="center">
  <img src="https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExMjI0MmlydDZ3aXFtNjU5M2g4OG1yeGlzcGxxNnpldW1idjY4bnVuaiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/Wn0xf87OpPZ4s8kGgP/giphy.gif" alt="Hacking GIF" width="800"/>
</div>

