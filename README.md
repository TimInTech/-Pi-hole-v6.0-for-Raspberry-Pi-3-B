# 🛡️ **Pi-hole v6.1.4: Advanced DNS Sinkhole for Raspberry Pi 3 B**  
*Complete guide for ARMv6 with Debian Bullseye and Whitelist Automation*  

![Pi-hole v6.1.4 Dashboard](https://github.com/user-attachments/assets/b0ad4d03-d118-4781-8dce-0a9956a978f2)  

🔗 **Official Resources**  
[Pi-hole GitHub](https://github.com/pi-hole/pi-hole) | [v6.1.4 Release Notes](https://github.com/pi-hole/pi-hole/releases/tag/v6.1.4)  
**Recommended Hardware**: [Raspberry Pi 4 Starter Kit](https://www.amazon.de/Raspberry-Starter-Kit-Netzteil-Geh%C3%A4use-K%C3%BChlk%C3%B6rper/dp/B0D1N3V2FF?tag=pinterestd00b-21) *(Compatible with Pi 3 B)*  

> **Looking for a comprehensive setup guide?**  
> Check out **[TimInTech/Pi-hole-Unbound-PiAlert-Setup](https://github.com/TimInTech/Pi-hole-Unbound-PiAlert-Setup)** for DNS over TLS and monitoring.

---

## 📋 **Table of Contents**  
1. [What's New in v6.1.4](#-whats-new-in-v614)  
2. [Hardware Requirements](#-hardware-requirements)  
3. [OS Installation](#-os-installation)  
4. [Pi-hole Setup](#-pi-hole-setup)  
5. [Whitelist Automation](#-whitelist-automation)  
6. [Maintenance](#-maintenance)  
7. [Troubleshooting](#-troubleshooting)  

---

## 🚀 **What's New in v6.1.4**  
### Critical Fixes & Improvements  
- **Gravity Update Fix**  
  Fixed crashes during `pihole -g` caused by empty shell variables  
- **Permission Enforcement**  
  Requires root/sudo for password changes and critical operations  
- **ARMv6 Optimization**  
  30% faster DNSSEC validation on Raspberry Pi 3 B  
- **Container Support**  
  Fixed issues in Docker image `2025.08.0`  

### Security Enhancements  
- **JWT Authentication**  
  Secure API access with token-based authentication  
- **HTTPS Enforcement**  
  Web interface now forces HTTPS connections  
- **TOML Configuration**  
  Centralized settings in `/etc/pihole/pihole.toml`  

---

## 🛠️ **Hardware Requirements**  
| Component | Specification |  
|-----------|---------------|  
| **Raspberry Pi** | 3 Model B (ARMv6) |  
| **OS** | [Raspberry Pi OS (Legacy) Lite](https://downloads.raspberrypi.org/raspios_lite_armhf/images/) |  
| **Storage** | 16GB+ MicroSD Card (Class 10 recommended) |  
| **Memory** | Minimum 1GB RAM (2GB for large blocklists) |  

**Why ARMv6?**  
Pi-hole v6.1.4 maintains full compatibility with legacy Raspberry Pi 3 B hardware running Debian Bullseye.

---

## 📥 **OS Installation**  
### Step 1: Flash the Image  
1. Use **Raspberry Pi Imager**  
2. Select OS:  
   ```plaintext
   Raspberry Pi OS (Other) → Raspberry Pi OS (Legacy) Lite (32-bit)
   ```  
3. Enable SSH: Click gear icon → Enable SSH → Set password  

### Step 2: First Boot & Update  
```bash
ssh pi@your-pi-ip

# Update system (critical for v6.1.4)
sudo apt update && sudo apt full-upgrade -y

# Install dependencies
sudo apt install curl git dnsutils -y
```

---

## ⚡ **Pi-hole v6.1.4 Setup**  
### Automated Install with Fixes  
```bash
# Run installer with OS check skip
curl -sSL https://install.pi-hole.net | \
  PIHOLE_SKIP_OS_CHECK=true sudo -E bash
```  

**Configuration Choices**:  
1. Upstream DNS: **Cloudflare (DoH)**  
2. Blocklist: **Default + StevenBlack**  
3. Web Admin: **Enabled**  
4. Query Logging: **Enabled**  

### Post-Install Steps  
```bash
# Secure web interface password
sudo pihole -a -p "your_strong_password"

# Enable auto-updates
sudo pihole -a updatechecker auto
```

---

## ⚙️ **Whitelist Automation**  
### 1. Create Whitelist File  
```bash
sudo nano /etc/pihole/whitelist.txt
```  
```plaintext
# Essential Services
alexa.amazon.com
device-metrics.us  
*.tuya.com

# Cloud Services
*.microsoft.com
*.googleapis.com
```

### 2. Auto-Update Script (v6.1.4 Optimized)  
```bash
#!/bin/bash
# Whitelist updater for v6.1.4+
pihole --regex --nuke
pihole -w --nuke
xargs -a /etc/pihole/whitelist.txt -I {} pihole -w {}
pihole restartdns reload
```

**Schedule Daily Updates**:  
```bash
sudo crontab -e
@daily /usr/local/bin/whitelist-updater.sh >/dev/null 2>&1
```

---

## 🔧 **Maintenance**  
### Update Management  
```bash
# Update Pi-hole core
sudo pihole -up

# Update blocklists
sudo pihole -g

# Update OS (weekly)
sudo apt update && sudo apt upgrade -y
```  

### Backup/Restore  
```bash
# Create backup
sudo pihole -a teleporter

# Restore from backup
sudo pihole -a -r /path/to/backup.tar.gz
```  

---

## 🚨 **Troubleshooting v6.1.4**  
| Issue | Solution |  
|-------|----------|  
| **Gravity Update Fail** | `sudo rm -f /etc/pihole/local.list; sudo pihole -g` |  
| **Web Interface Crash** | `sudo systemctl restart pihole-admin` |  
| **"Database Locked"** | `sudo systemctl restart pihole-FTL` |  
| **DNS Failure** | `sudo pihole restartdns reload` |  
| **Permission Errors** | `sudo pihole -a -p` (reset password) |  

### Diagnostic Commands  
```bash
# Check service status
pihole status

# Test DNS resolution
dig @127.0.0.1 pi-hole.net +short

# View live logs
pihole -t -ex
```

---

## 🐋 **Docker Installation**  
```bash
docker run -d --name pihole \
  -p 53:53/tcp -p 53:53/udp \
  -p 80:80 \
  -e TZ="Europe/Berlin" \
  -e WEBPASSWORD="secure_pw" \
  -e FTLCONF_MAXDBDAYS=30 \
  -v ~/pihole-data:/etc/pihole \
  --restart=unless-stopped \
  pihole/pihole:2025.08.0
```

> **Note**: Raspberry Pi 3 B users should use `--platform linux/arm/v6` flag

---

## 📊 **Performance Tips for Pi 3 B**  
1. **Blocklist Size**  
   Keep under 500,000 domains (use `pihole -b -f` to flush)  
2. **Log Retention**  
   Reduce to 7 days: `sudo sqlite3 /etc/pihole/pihole-FTL.db "UPDATE config SET value=7 WHERE key='maxlogdays';"`  
3. **Memory Optimization**  
   Add to `/etc/pihole/pihole.toml`:  
   ```toml
   [FTL]
   max_cache_entries = 100000
   ```

**Expected Performance**:  
- 50-60k daily queries on Pi 3 B  
- 15-20ms average response time  
