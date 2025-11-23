# Pi-hole v6.x Guide for Raspberry Pi 3 B
*Complete guide for running Pi-hole v6 on a Raspberry Pi 3 B with Debian/Raspberry Pi OS and secure DNS configuration.*

<p align="center">
  <a href="https://github.com/TimInTech/-Pi-hole-v6.0-for-Raspberry-Pi-3-B/stargazers"><img alt="GitHub Stars" src="https://img.shields.io/github/stars/TimInTech/-Pi-hole-v6.0-for-Raspberry-Pi-3-B?style=flat&color=yellow"></a>
  <a href="https://github.com/TimInTech/-Pi-hole-v6.0-for-Raspberry-Pi-3-B/network/members"><img alt="GitHub Forks" src="https://img.shields.io/github/forks/TimInTech/-Pi-hole-v6.0-for-Raspberry-Pi-3-B?style=flat&color=blue"></a>
  <a href="LICENSE.md"><img alt="License" src="https://img.shields.io/github/license/TimInTech/-Pi-hole-v6.0-for-Raspberry-Pi-3-B?style=flat"></a>
  <a href="https://github.com/TimInTech/-Pi-hole-v6.0-for-Raspberry-Pi-3-B/releases/latest"><img alt="Latest Release" src="https://img.shields.io/github/v/release/TimInTech/-Pi-hole-v6.0-for-Raspberry-Pi-3-B?include_prereleases&style=flat"></a>
  <a href="https://buymeacoffee.com/timintech"><img alt="Buy Me A Coffee" src="https://img.shields.io/badge/Buy%20Me%20A%20Coffee-FFDD00?logo=buymeacoffee&logoColor=000&labelColor=grey&style=flat"></a>
</p>

![Pi-hole v6.1.4 Dashboard](https://github.com/user-attachments/assets/b0ad4d03-d118-4781-8dce-0a9956a978f2)

## Project Overview
Step-by-step instructions for deploying Pi-hole v6.1.4 on a Raspberry Pi 3 B with Debian 12 / Raspberry Pi OS (Legacy) Lite, including secure DNS choices, whitelist automation, and performance tuning for ARMv6 hardware.

## Quick Links
- Main guide: `README.md` (this document)
- Troubleshooting: [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- Quickstart: [Quickstart](#quickstart)
- Installation: [Installation & Setup](#installation--setup)
- Troubleshooting summary: [Troubleshooting](#troubleshooting)

## What It Is
- A Pi-hole v6.1.4 guide tailored for Raspberry Pi 3 B / ARMv6 on Debian 12 / Raspberry Pi OS.
- Focus on secure DNS (e.g., Cloudflare DoH), centralized `pihole.toml` configuration, and lightweight defaults for constrained hardware.
- Includes whitelist automation, maintenance routines, and notes on Docker usage where applicable.

## Requirements
### Hardware
- Raspberry Pi 3 Model B (tested); Pi 4 is compatible but not the focus.
- 16 GB+ microSD card (Class 10 recommended), reliable power, and network connectivity (wired preferred).

### Software
- Debian 12 / Raspberry Pi OS (Legacy) Lite (32-bit).
- Pi-hole v6.1.4 core/FTL/web.
- Optional: Docker (for containerized deployments).

### Network
- Static or DHCP-reserved IP for the Pi.
- Router/DHCP handing out the Pi-hole IP as primary DNS.
- Outbound DNS allowed to upstream resolvers if using DoH/DoT clients.

## Installation & Setup
### 1) Prepare the OS
1. Flash Raspberry Pi OS (Legacy) Lite (32-bit) with Raspberry Pi Imager.
2. Enable SSH in Imager options (or drop an empty `ssh` file on the boot partition).
3. First boot and update:
   ```bash
   ssh pi@192.168.1.10
   sudo apt update && sudo apt full-upgrade -y
   sudo apt install -y curl git dnsutils
   ```

### 2) Install Pi-hole v6.1.4
```bash
curl -sSL https://install.pi-hole.net | PIHOLE_SKIP_OS_CHECK=true sudo -E bash
```
Recommended installer choices:
- Upstream DNS: Cloudflare (DoH) or a trusted resolver of your choice.
- Blocklists: Defaults or defaults plus StevenBlack for broader coverage.
- Web admin: Enabled.
- Query logging: Enabled with sensible retention.

### 3) Secure and configure
```bash
# Set/rotate web interface password
sudo pihole -a -p

# Enable automatic update checks
sudo pihole -a updatechecker auto
```

Centralize key options in `/etc/pihole/pihole.toml`:
```toml
[dns]
upstreams = [
  "1.1.1.1",
  "1.0.0.1"
]

[FTL]
max_cache_entries = 100000

[database]
maxDBdays = 7
```

### 4) Whitelist automation (optional)
Create a whitelist file:
```bash
sudo nano /etc/pihole/whitelist.txt
```
Example entries:
```plaintext
# Essential services
alexa.amazon.com
device-metrics.us
*.tuya.com

# Cloud services
*.microsoft.com
*.googleapis.com
```

Automation script (place at `/usr/local/bin/whitelist-updater.sh`, then `sudo chmod +x /usr/local/bin/whitelist-updater.sh`):
```bash
#!/bin/bash
# Whitelist updater for Pi-hole v6.1.4+
set -euo pipefail

while read -r domain; do
  [[ -z "$domain" || "$domain" =~ ^# ]] && continue
  pihole allow "$domain"
done < /etc/pihole/whitelist.txt

pihole reloaddns
```
Schedule daily:
```bash
sudo crontab -e
@daily /usr/local/bin/whitelist-updater.sh >/dev/null 2>&1
```

## Status
- Tested on Raspberry Pi 3 B with Raspberry Pi OS (Legacy) Lite / Debian 12.
- Focused on Pi-hole v6.1.4 core/FTL/web stack with ARMv6 performance tweaks.

## Quickstart
```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y curl git dnsutils
curl -sSL https://install.pi-hole.net | PIHOLE_SKIP_OS_CHECK=true sudo -E bash
sudo pihole -a -p
pihole status
dig @127.0.0.1 pi-hole.net +short
```

## Technologies & Dependencies
<p>
  <img src="https://img.shields.io/badge/Pi--hole-v6.1.4-BA1A1A?logo=pihole&logoColor=white&labelColor=000000" alt="Pi-hole v6.1.4">
  <img src="https://img.shields.io/badge/Debian-12%20%7C%20Raspberry%20Pi%20OS-A81D33?logo=debian&logoColor=white" alt="Debian / Raspberry Pi OS">
  <img src="https://img.shields.io/badge/Bash-Scripting-4EAA25?logo=gnubash&logoColor=white" alt="Bash">
  <img src="https://img.shields.io/badge/systemd-service%20management-244A87?logo=systemd&logoColor=white" alt="systemd">
  <img src="https://img.shields.io/badge/Docker-optional-2496ED?logo=docker&logoColor=white" alt="Docker">
</p>

## Features
- Pi-hole v6.1.4 guidance with centralized TOML configuration for DNS, FTL, and database settings.
- Highlights: gravity update crash fixes, enforced sudo for sensitive actions, faster DNSSEC on ARMv6, Docker `2025.08.0` compatibility, HTTPS-first admin UI with JWT-secured API.
- Whitelist automation using `pihole allow`, plus DNS reload for clean application.
- Performance tips for Pi 3 B: tuned cache sizing, log retention, and lightweight blocklist choices.
- Maintenance routines for gravity updates, OS patching, and backup via Teleporter.

## Usage / Typical Scenarios
- **Home network DNS sinkhole:** Point router DHCP DNS to the Pi-hole IP; confirm with `pihole status`.
- **Diagnostics:**  
  ```bash
  pihole tail                   # live logs
  dig @127.0.0.1 example.com +short
  grep example.com /var/log/pihole/pihole.log
  ```
- **Maintenance:**  
  ```bash
  sudo pihole -up               # update core/FTL/web
  sudo pihole -g                # gravity refresh (pihole updateGravity)
  sudo apt update && sudo apt upgrade -y
  ```
- **Docker (optional):**  
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
  For Raspberry Pi 3 B, include `--platform linux/arm/v6` if required by your Docker host.

## Troubleshooting
- Full guide: [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- Common issues covered: DNS not blocking/resolving, slow queries, local domain handling, blocklist/whitelist sync, IPv6/network conflicts, and external DNS blocking tips.

## Contributing & License
- Issues and pull requests are welcome; keep instructions reproducible for Raspberry Pi 3 B users.
- Released under the MIT License. See [LICENSE](LICENSE.md) for details.
