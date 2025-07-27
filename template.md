# TryHackMe [ROOM_NAME] - Complete Walkthrough

![TryHackMe](https://img.shields.io/badge/TryHackMe-[ROOM_NAME]-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-[EASY/MEDIUM/HARD]-[green/orange/red]?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-[CATEGORY]-blue?style=for-the-badge)

## 📋 Table of Contents
- [Overview](#overview)
- [Room Information](#room-information)
- [Initial Setup](#initial-setup)
- [Reconnaissance](#reconnaissance)
- [Enumeration](#enumeration)
- [Initial Access](#initial-access)
- [Post-Exploitation](#post-exploitation)
- [Privilege Escalation](#privilege-escalation)
- [Flags](#flags)
- [Questions & Answers](#questions--answers)
- [Alternative Methods](#alternative-methods)
- [Key Learnings](#key-learnings)
- [Tools Used](#tools-used)
- [References](#references)

## 🎯 Overview
Brief description of what this room teaches and main objectives:
- **Primary Skills:** [List main skills learned]
- **Technologies:** [List technologies involved]
- **Attack Vectors:** [List main attack methods]
- **Difficulty Justification:** [Why this difficulty level]

## 📊 Room Information
- **Room URL:** [TryHackMe room link]
- **Creator:** [Room creator if known]
- **Release Date:** [If known]
- **Estimated Time:** [Your completion time]
- **Prerequisites:** [Required knowledge/rooms]

## 🚀 Initial Setup

### 🌐 Environment Preparation
```bash
# Create working directory
mkdir [room_name]
cd [room_name]
mkdir {nmap,gobuster,exploits,flags,notes}

# Set target IP variable
export TARGET_IP=[MACHINE_IP]
```

### 📡 Network Configuration
```bash
# Verify connectivity
ping -c 3 $TARGET_IP

# Add to /etc/hosts if needed
echo "$TARGET_IP [hostname]" | sudo tee -a /etc/hosts
```

## 🔍 Reconnaissance

### 🌐 Port Scanning
```bash
# Quick scan
nmap -T4 -p- --min-rate=1000 $TARGET_IP

# Detailed scan
nmap -sC -sV -p [ports] -oN nmap/detailed_scan.txt $TARGET_IP

# UDP scan (if needed)
sudo nmap -sU --top-ports 1000 $TARGET_IP
```

### 📊 Scan Results
| Port | Service | Version | Notes |
|------|---------|---------|-------|
| [port] | [service] | [version] | [observations] |

### 🔍 Service Enumeration
```bash
# Additional service-specific scans
# Example for HTTP:
nmap --script http-enum -p 80 $TARGET_IP

# Example for SMB:
nmap --script smb-enum-* -p 445 $TARGET_IP
```

## 🕸️ Enumeration

### 📁 Web Directory Enumeration
```bash
# Gobuster
gobuster dir -u http://$TARGET_IP -w /usr/share/seclists/Discovery/Web-Content/big.txt -o gobuster/directories.txt

# Alternative tools
# dirb http://$TARGET_IP
# ffuf -w wordlist.txt -u http://$TARGET_IP/FUZZ
```

### 🔍 Technology Stack
- **Web Server:** [Apache/Nginx/IIS version]
- **Backend:** [PHP/Python/Node.js/etc]
- **Database:** [MySQL/PostgreSQL/etc]
- **CMS/Framework:** [WordPress/Drupal/etc]

### 📋 Discovered Endpoints
| Path | Status | Size | Notes |
|------|---------|------|-------|
| [/path] | [200/403/etc] | [size] | [observations] |

## 🚪 Initial Access

### 🎯 Attack Vector
**Method Used:** [Describe the method]

### 💥 Exploitation Steps
```bash
# Step 1: [Description]
[command]

# Step 2: [Description]
[command]

# Step 3: [Description]
[command]
```

### 🐚 Shell Stabilization
```bash
# Upgrade shell (if needed)
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
# Ctrl+Z
stty raw -echo; fg
```

### 👤 Initial User Context
- **Username:** [username]
- **UID/GID:** [uid/gid info]
- **Groups:** [group memberships]
- **Home Directory:** [home path]
- **Shell:** [shell type]

## 🔍 Post-Exploitation

### 🗂️ System Information
```bash
# Basic system info
uname -a
cat /etc/os-release
whoami && id
```

### 📁 File System Exploration
```bash
# Interesting directories
ls -la /home/
ls -la /var/www/
ls -la /opt/
ls -la /tmp/

# Configuration files
find /etc -name "*.conf" -readable 2>/dev/null
```

### 🔐 Credential Hunting
```bash
# Common credential locations
cat /etc/passwd
cat /etc/shadow 2>/dev/null
grep -r "password" /var/www/ 2>/dev/null
find / -name "*.sql" -o -name "*.db" 2>/dev/null
```

### 🌐 Network Analysis
```bash
# Network connections
netstat -tulpn
ss -tulpn

# Network interfaces
ip addr show
ifconfig
```

## 🚀 Privilege Escalation

### 🔍 Enumeration
```bash
# Automated enumeration
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh

# Manual checks
sudo -l
find / -perm -4000 2>/dev/null
crontab -l
systemctl list-timers
```

### 🎯 Privilege Escalation Vector
**Method:** [Describe the method used]

### 💥 Exploitation
```bash
# Exploitation commands
[detailed steps]
```

### 👑 Root Verification
```bash
whoami
id
cat /etc/shadow
```

## 🏆 Flags

### 📍 Flag Locations
- **User Flag:** `[flag_value]` - Location: `[path]`
- **Root Flag:** `[flag_value]` - Location: `[path]`
- **Other Flags:** 
  - Flag 3: `[value]` - `[location]`
  - Flag 4: `[value]` - `[location]`

## ❓ Questions & Answers

### Task 1: [Task Name]
**Q1:** [Question text]  
**A1:** `[answer]`  
**Method:** [How you found it]

**Q2:** [Question text]  
**A2:** `[answer]`  
**Method:** [How you found it]

### Task 2: [Task Name]
**Q1:** [Question text]  
**A1:** `[answer]`  
**Method:** [How you found it]

## 🔄 Alternative Methods

### 🎯 Alternative Attack Vectors
1. **Method 1:** [Description]
   ```bash
   # Commands
   ```

2. **Method 2:** [Description]
   ```bash
   # Commands
   ```

### 🚀 Alternative Privilege Escalation
- **Vector 1:** [Description and commands]
- **Vector 2:** [Description and commands]

## 📚 Key Learnings

### 🔍 Technical Skills
- **[Skill 1]:** [What you learned]
- **[Skill 2]:** [What you learned]
- **[Skill 3]:** [What you learned]

### 🛡️ Security Concepts
- **[Concept 1]:** [Understanding gained]
- **[Concept 2]:** [Understanding gained]

### 🎯 Methodology Improvements
- **[Improvement 1]:** [What you'll do differently]
- **[Improvement 2]:** [What you'll do differently]

### ⚠️ Common Pitfalls
- **[Pitfall 1]:** [What to avoid and why]
- **[Pitfall 2]:** [What to avoid and why]

## 🛠️ Tools Used

### 🔍 Reconnaissance & Enumeration
- `nmap` - [Specific usage]
- `gobuster` - [Specific usage]
- `[tool]` - [Specific usage]

### 💥 Exploitation
- `[exploit_tool]` - [Purpose]
- `[payload]` - [Purpose]

### 🚀 Post-Exploitation
- `linpeas.sh` - [Usage]
- `[tool]` - [Usage]

### 🔧 Utilities
- `netcat` - [Usage]
- `python3` - [Usage]
- `[tool]` - [Usage]

## 📖 References

### 🔗 Helpful Resources
- **[Resource 1]:** [URL] - [Description]
- **[Resource 2]:** [URL] - [Description]
- **[CVE/Exploit]:** [Details]

### 📚 Learning Materials
- **[Tutorial/Guide]:** [URL]
- **[Documentation]:** [URL]

### 🛠️ Tool Documentation
- **[Tool Name]:** [Documentation URL]

## 🎯 Attack Chain Summary
```
1. [Step 1] → [Result]
2. [Step 2] → [Result]
3. [Step 3] → [Result]
4. [Step 4] → [Result]
5. [Step 5] → [Complete Compromise]
```

## 📋 Personal Notes

### 💭 Thought Process
- [Your thinking during the challenge]
- [Decisions made and why]
- [Roadblocks encountered]

### ⏱️ Time Breakdown
- **Reconnaissance:** [time]
- **Initial Access:** [time]
- **Privilege Escalation:** [time]
- **Documentation:** [time]
- **Total:** [total time]

### 🎯 Difficulty Assessment
**Personal Rating:** [X/10]  
**Reasoning:** [Why this rating]

### 🔄 Areas for Improvement
- [What you could have done better]
- [Skills to develop further]
- [Tools to learn]

---

> **💡 Pro Tip:** [Your key insight or tip for others attempting this room]

**Completion Date:** [Date]  
**Author:** Your Security Notes  
**Room Status:** ✅ Completed  
**Tags:** `#tryhackme` `#[tag1]` `#[tag2]` `#[tag3]`

---

## 📝 Template Usage Instructions

1. **Replace all bracketed placeholders** with actual content
2. **Remove unused sections** if not applicable
3. **Add additional sections** as needed for specific room types
4. **Update badges** with correct difficulty colors:
   - Easy: `green`
   - Medium: `orange` 
   - Hard: `red`
5. **Customize categories** as needed:
   - Web Exploitation
   - Privilege Escalation
   - Boot2Root
   - Forensics
   - Cryptography
   - Reverse Engineering
   - etc.
