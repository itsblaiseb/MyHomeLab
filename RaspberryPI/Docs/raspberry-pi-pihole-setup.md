# 🧱 Raspberry Pi Ad-Blocking Server with Pi-hole

This document records my setup of a **Raspberry Pi 4** as a **network-wide ad blocker** using **Pi-hole**, with plans to later extend it into a **WireGuard VPN server**.

---

## 🧰 Hardware & Environment

| Component | Details |
|----------|---------|
| **Device** | Raspberry Pi 4 Model B |
| **OS** | Raspberry Pi OS (Debian 13 “Trixie”, 64-bit, aarch64) |
| **Hostname** | `blaise` |
| **User** | `administrator` |
| **Connection** | Ethernet (LAN only) |
| **IP Address** | `192.168.1.101` |
| **Router Gateway** | `192.168.1.1` |
| **Wi‑Fi** | Disabled / blocked by rfkill (LAN only) |

---

## ⚙️ 1. Initial System Setup

### SSH Access
From Windows (PowerShell/CMD):
```powershell
ssh administrator@192.168.1.101
```

If prompted about authenticity, type `yes`.

### Update & Upgrade Packages
```bash
sudo apt update && sudo apt upgrade -y
```

Reboot after upgrading:
```bash
sudo reboot
```

Reconnect with SSH once it comes back up.

---

## 🌍 2. Configure Networking (Bookworm/Trixie via NetworkManager)

Modern Raspberry Pi OS releases (Bookworm/Trixie) use **NetworkManager** instead of `dhcpcd`.

Check your interface:
```bash
ip addr
```
You should see `eth0` with `inet 192.168.1.101/24`.

Set a **static IP** and DNS (self-referencing Pi-hole):
```bash
sudo nmcli con mod "Wired connection 1" ipv4.addresses 192.168.1.101/24 ipv4.gateway 192.168.1.1 ipv4.dns "127.0.0.1" ipv4.method manual
sudo nmcli con down "Wired connection 1" && sudo nmcli con up "Wired connection 1"
```

Verify connections:
```bash
nmcli con show
```

---

## 🧩 3. Install Pi-hole

Run the official installer:
```bash
curl -sSL https://install.pi-hole.net | bash
```

### Installer Configuration
| Prompt | Choice |
|-------|--------|
| Interface | `eth0` |
| Static IP | `192.168.1.101/24` |
| Upstream DNS | `8.8.8.8` (Google) |
| Web Admin Interface | Yes |
| Web Server (lighttpd) | Yes |
| Query Logging | Yes |
| Privacy Mode | `0 - Show everything` |

At the end, the script will display:
```
[i] View the web interface at http://pi.hole/admin or http://192.168.1.101/admin
[i] Web Interface password: <initial_password_shown_by_installer>
```
Change the admin password anytime:
```bash
sudo pihole setpassword
```

---

## 🌐 4. Access the Web Dashboard

Open:
```
http://192.168.1.101/admin
```
or
```
http://pi.hole/admin
```

Log in with the password above (or the one you just set).

---

## 🧱 5. Verify DNS Configuration

Ensure the Pi itself uses Pi-hole for DNS:
```bash
dig doubleclick.net
```
Expected:
```
SERVER: 127.0.0.1#53(127.0.0.1)
doubleclick.net.  IN  A  0.0.0.0
```

If you see `0.0.0.0`, Pi-hole is actively blocking ads.

---

## 📦 6. Update Blocklists (Gravity)

Rebuild the blocklist database:
```bash
sudo pihole -g
```
Sample output:
```
[i] Target: https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
[✓] Parsed 101221 exact domains
[✓] Optimizing database
[✓] Done.
```

### Optional: Add More Blocklists
In the web UI (**Settings → Blocklists → Add new list**):
```
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
https://blocklistproject.github.io/Lists/ads.txt
https://blocklistproject.github.io/Lists/tracking.txt
```
Then update again:
```bash
sudo pihole -g
```

---

## 🧪 7. Test Ad Blocking

```bash
dig doubleclick.net
```
Expected:
```
doubleclick.net. IN A 0.0.0.0
```
Ad domain successfully blocked ✅

---

## 📡 8. Configure Network to Use Pi-hole

In your router’s admin (usually `http://192.168.1.1`), set **LAN/DHCP DNS**:
```
Primary DNS: 192.168.1.101
Secondary DNS: 1.1.1.1 (optional fallback)
```
Save and reboot the router. Clients will pick up Pi-hole automatically.

---

## 🔄 9. Maintenance

### Update Pi-hole
```bash
pihole -up
```

### Refresh Blocklists
```bash
sudo pihole -g
```

### Check Status
```bash
pihole status
```
Expected:
```
[✓] DNS service is running
[✓] Pi-hole blocking is enabled
```

---

## 🧰 10. Useful Commands

| Task | Command |
|------|--------|
| Live logs | `pihole -t` |
| Change web password | `sudo pihole setpassword` |
| Update OS | `sudo apt update && sudo apt upgrade -y` |
| Query lists for a domain | `pihole -q example.com` |
| Restart DNS service | `sudo systemctl restart pihole-FTL` |

---

## ✅ Current State

| Component | Status |
|----------|--------|
| **Pi-hole DNS** | ✅ Active |
| **Ad blocking** | ✅ Working (`doubleclick.net → 0.0.0.0`) |
| **Web dashboard** | ✅ `http://192.168.1.101/admin` |
| **Static IP** | ✅ NetworkManager |
| **Wi‑Fi** | 🚫 Blocked (LAN only) |
| **Next phase** | Integrate **WireGuard VPN** |

---

## 📘 Next Step

**WireGuard VPN Integration** (coming next): route remote device traffic through the Pi so it benefits from Pi-hole filtering anywhere.
