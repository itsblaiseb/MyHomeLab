# Phase 1 – Secure Base Setup for Raspberry Pi (Homelab VPN + Pi-hole)

This document describes the foundational security configuration for the Raspberry Pi homelab server (`homelab`), used for VPN and Pi-hole services.  
All steps are focused on minimizing attack surface and ensuring safe remote access.

---

## System Information
- **Hostname:** `homelab`
- **OS:** Raspberry Pi OS Lite (64-bit)
- **Kernel:** Linux 6.12.47+rpt-rpi-v8 (aarch64)
- **Primary Interface:** `eth0`
- **Static IP:** `192.168.77.10`
- **Router/Gateway:** `192.168.77.1`

---

## Step 1 – Static IP Assignment

Configured via **NetworkManager** (`nmcli`):

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.addresses 192.168.77.10/24
sudo nmcli connection modify "Wired connection 1" ipv4.gateway 192.168.77.1
sudo nmcli connection modify "Wired connection 1" ipv4.dns "192.168.77.1,8.8.8.8"
sudo nmcli connection modify "Wired connection 1" ipv4.method manual
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

**Result:**  
Raspberry Pi now uses the static IP `192.168.77.10`.

---

## Step 2 – SSH Hardening

### Generate SSH Keys (on Windows PC)
```powershell
ssh-keygen -t ed25519
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh yourusername@192.168.77.10 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### Configure SSH Daemon
Edit `/etc/ssh/sshd_config`:
```
Port 2256
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart SSH:
```bash
sudo systemctl restart ssh
```

Connect using:
```powershell
ssh -p 2256 yourusername@192.168.77.10
```

**Result:**  
SSH is accessible only via public-key authentication on port 2256.

---

## Step 3 – Firewall (UFW)

### Install and Configure
```bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2256/tcp
sudo ufw enable
sudo ufw status
```

**Result:**  
- Incoming connections: **denied by default**  
- Outgoing: **allowed**  
- SSH (port 2256) is explicitly permitted  

---

## Step 4 – Fail2Ban Intrusion Protection

### Install and Enable
```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### Local Configuration
Create `/etc/fail2ban/jail.local`:
```
[sshd]
enabled = true
port    = 2256
filter  = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 60m
findtime = 5m
```

Restart Fail2Ban:
```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd
```

**Result:**  
- Protects SSH (port 2256) from brute-force attacks  
- Bans IPs after 3 failed attempts within 5 minutes  
- Ban duration: 1 hour  

---

## Summary of Security Changes

| Feature | Configuration |
|----------|----------------|
| **Static IP** | `192.168.77.10` |
| **SSH Port** | `2256` |
| **Firewall** | Default deny incoming / allow outgoing |
| **Authentication** | Public key only |
| **Fail2Ban** | Enabled (3 retries, 1 hour ban) |

---

**Phase 1 Complete:**  
Your Raspberry Pi now has a hardened baseline — secure SSH access, firewall protection, and automated intrusion prevention.

Next up → **Phase 2 – Install Pi-hole & Unbound**, followed by **Phase 3 – VPN (WireGuard) Setup**.
