# 🛡️ **Pi-hole v6.0: Advanced DNS Sinkhole for Raspberry Pi 3 B**  
*Complete guide for ARMv6 with Debian Bullseye and Whitelist Automation*  

![Pi-hole v6.0 Dashboard](https://github.com/user-attachments/assets/b0ad4d03-d118-4781-8dce-0a9956a978f2)  

🔗 **Official Resources**  
[Pi-hole GitHub](https://github.com/pi-hole/pi-hole) | [v6 Migration Guide](https://docs.pi-hole.net/docker/upgrading/v5-v6/)  
**Recommended Hardware**: [Raspberry Pi 4 Starter Kit](https://www.amazon.de/Raspberry-Starter-Kit-Netzteil-Geh%C3%A4use-K%C3%BChlk%C3%B6rper/dp/B0D1N3V2FF?tag=pinterestd00b-21) *(Compatible with Pi 3 B)*  

---

## 📋 **Table of Contents**  
1. [Hardware Requirements](#-hardware-requirements)  
2. [OS Installation](#-os-installation)  
3. [Pi-hole Setup](#-pi-hole-setup)  
4. [Whitelist Automation](#-whitelist-automation)  
5. [Maintenance](#-maintenance)  

🛠️ Pi-hole v6 - Troubleshooting Guide
6. [Troubleshooting](TROUBLESHOOTING.md)  

---

## 🛠️ **Hardware Requirements**  
| Component | Specification |  
|-----------|---------------|  
| **Raspberry Pi** | 3 Model B (ARMv6) |  
| **OS** | [Raspberry Pi OS (Legacy) Lite (32-bit)](https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2023-12-11/) |  
| **Storage** | 16GB+ MicroSD Card |  
| **Power Supply** | 5V/2.5A |  

**Why ARMv6?**  
Pi-hole v6 requires Debian Bullseye (11), which is only fully compatible with the legacy OS on Raspberry Pi 3 B.  

---

## 📥 **OS Installation**  
### Step 1: Flash the Image  
1. Use **Raspberry Pi Imager**  
2. Select OS:  
   ```plaintext
   Raspberry Pi OS (Other) → Raspberry Pi OS (Legacy) Lite (32-bit)
   ```  
3. Enable SSH: Click the gear icon → Enable SSH → Set password  

### Step 2: First Boot  
```bash
# Connect via SSH
ssh pi@192.168.1.100 # Replace with your IP
Password: raspberry

# Update system
sudo apt update && sudo apt full-upgrade -y
```

---

## ⚡ **Pi-hole Setup**  
### Automated Install  
```bash
# Run installer (skip OS check)
curl -sSL https://install.pi-hole.net | PIHOLE_SKIP_OS_CHECK=true sudo -E bash
```  

**Key Choices**:  
- Upstream DNS: **Google (ECS + DNSSEC)**  
- Web Interface: **Enabled**  
- Logging: **Enabled**  

---

## ⚙️ **Whitelist Automation**  
### 1. Create Your Whitelist  
1. Create a `whitelist.txt` file:  
   ```plaintext
   # Example Whitelist
   alexa.amazon.com
   device-metrics.us  
   *.tuya.com
   ```  
2. Host it privately:  
   - **GitHub Private Repo**: Upload and use the raw URL  
   - **Local Server**: Use `python3 -m http.server 8000`  

### 2. Auto-Update Script  
```bash
sudo nano /usr/local/bin/whitelist-updater.sh
```  
```bash
#!/bin/bash
WHITELIST_URL="https://raw.githubusercontent.com/YourUsername/PrivateRepo/main/whitelist.txt"

wget -q -O /tmp/whitelist.txt "$WHITELIST_URL"
pihole -w --nuke
xargs -a /tmp/whitelist.txt -I {} pihole -w {}
pihole restartdns
```  

**Schedule Daily Updates**:  
```bash
sudo crontab -e
# Add:
@daily /usr/local/bin/whitelist-updater.sh >/dev/null 2>&1
```  

---

## 🔧 **Maintenance**  
### 1. Regular Updates  
```bash
# Update Pi-hole
pihole -up

# Update OS
sudo apt update && sudo apt upgrade -y
```  

### 2. Backup/Restore  
```bash
# Export settings
pihole -a -t

# Restore
pihole -a -r /path/to/backup.tar.gz
```  

---

## 🚨 **Troubleshooting**  
| Issue | Solution |  
|-------|----------|  
| **DNS Failure** | `pihole restartdns` |  
| **Web Interface Crash** | `sudo systemctl restart lighttpd` |  
| **"Database Locked"** | `sudo systemctl restart pihole-FTL` |  

---
