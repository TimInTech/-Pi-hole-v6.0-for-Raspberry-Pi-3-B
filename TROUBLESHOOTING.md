# 🛠️ Pi-hole v6 - Troubleshooting Guide

Solutions to common issues with Pi-hole v6 on Debian 12 / Raspberry Pi OS (Legacy) Lite, focusing on Raspberry Pi 3 B and ARMv6 hardware.

---

## 📌 1. DNS Resolution Issues

### 🔹 Pi-hole is not blocking ads
1. Confirm clients use Pi-hole:
   ```bash
   nslookup pi.hole
   ```
2. Reload DNS after changes:
   ```bash
   pihole reloaddns
   ```
3. Refresh blocklists:
   ```bash
   pihole -g   # pihole updateGravity
   ```

### 🔹 Slow site loads or long query times
1. Measure latency:
   ```bash
   dig google.com @127.0.0.1 -p 5335
   ```
2. Verify upstream resolver responsiveness (Unbound/DoH/DoT as applicable).
3. Tune cache sizes in `/etc/pihole/pihole.toml` if running on constrained hardware.

### 🔹 Local domains are not resolving
Define hosts in `pihole.toml` and reload DNS:
```toml
# /etc/pihole/pihole.toml
[dns]
hosts = [
  "192.168.1.100 nas.local",
  "192.168.1.20 server.local"
]
```
```bash
pihole reloaddns
```

---

## 🔧 2. Whitelisting & Blocklist Issues

### 🔹 A website is blocked even after whitelisting
1. Check status:
   ```bash
   pihole query example.com   # same as pihole -q
   ```
2. Allow explicitly:
   ```bash
   pihole allow example.com
   pihole reloaddns
   ```

### 🔹 Blocklists are not updating
1. Trigger update:
   ```bash
   pihole -g   # pihole updateGravity
   ```
2. Review the log:
   ```bash
   cat /var/log/pihole/pihole_updateGravity.log
   ```

---

## 🌍 3. IPv6 & Network Issues

### 🔹 IPv6 queries are not being blocked
1. Test AAAA resolution:
   ```bash
   dig AAAA example.com @127.0.0.1 -p 5335
   ```
2. Ensure IPv6 is enabled and routed correctly on your network.

### 🔹 Some devices bypass Pi-hole
1. Ensure DHCP hands out the Pi-hole IP as primary DNS.
2. Block external DNS at the router/firewall (recommended) or on the Pi if acting as a gateway:
   ```bash
   sudo iptables -A OUTPUT -p udp --dport 53 -j REJECT
   sudo iptables -A OUTPUT -p tcp --dport 53 -j REJECT
   ```
3. Define upstreams in `pihole.toml` instead of legacy CLI:
   ```toml
   # /etc/pihole/pihole.toml
   [dns]
   upstreams = [
     "1.1.1.1",
     "1.0.0.1"
   ]
   ```
   ```bash
   pihole reloaddns
   ```

---

## 📈 4. Performance & Optimization

### 🔹 Pi-hole uses too much memory or storage
1. Keep blocklists lean via the web UI (Group Management → Adlists) or CLI equivalents; avoid excessively large lists on Pi 3 B.
2. Move database retention to `pihole.toml`:
   ```toml
   # /etc/pihole/pihole.toml
   [database]
   maxDBdays = 7
   DBinterval = 60.0
   ```

### 🔹 Reduce Unbound CPU usage (if using Unbound)
1. Optimize `unbound.conf`:
   ```bash
   num-threads: 1
   msg-cache-size: 4m
   rrset-cache-size: 8m
   cache-max-ttl: 86400
   cache-min-ttl: 3600
   ```

---

## 🛑 5. Debugging & Logs

### 🔹 Live view and query checks
```bash
pihole tail
dig @127.0.0.1 example.com +short
grep example.com /var/log/pihole/pihole.log
```

### 🔹 Service health
```bash
sudo systemctl status pihole-FTL.service
sudo journalctl -u pihole-FTL.service -n 200
```

### 🔹 Generate a debug report
```bash
pihole debug   # same as pihole -d
```

---

## 📝 6. Reporting Issues
If problems persist, capture a debug token and include relevant log excerpts when opening an issue.

---

Tested with Pi-hole v6.x on Raspberry Pi 3 B using Debian 12 / Raspberry Pi OS (Legacy) Lite.
