# Pi-hole + Unbound Recursive DNS Integration

This phase configures Pi-hole to use **Unbound** as a local recursive DNS resolver, ensuring fully private, DNSSEC‑validated lookups with no reliance on external resolvers.

---

## 1. Install Unbound

```bash
sudo apt install unbound -y
unbound -V
```
 Expected: version 1.17 or newer (v1.22.0 confirmed).

---

## 2. Configure Unbound for Pi-hole

Create configuration file:

```bash
sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
```

Add:

```bash
server:
    verbosity: 0
    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes
    do-ip6: no
    root-hints: /var/lib/unbound/root.hints
    harden-glue: yes
    harden-dnssec-stripped: yes
    edns-buffer-size: 1232
    prefetch: yes
    num-threads: 1
    so-reuseport: yes
    minimal-responses: yes
    cache-min-ttl: 3600
    cache-max-ttl: 86400

    private-address: 192.168.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10
```

Save (`Ctrl + O`, `Enter`, `Ctrl + X`).

---

## 3. Download Root Hints

```bash
sudo mkdir -p /var/lib/unbound
sudo wget -O /var/lib/unbound/root.hints https://www.internic.net/domain/named.root
sudo systemctl restart unbound
```

---

## 4. Verify Local Resolver

```bash
dig @127.0.0.1 -p 5335 example.com
```
 Should return `NOERROR` and a valid IP address.

---

## 5. Integrate with Pi-hole (v6 TOML method)

Pi-hole v6 uses `/etc/pihole/pihole.toml` instead of the legacy `setdns` command.

Edit the file:

```bash
sudo nano /etc/pihole/pihole.toml
```

Find the `[dns]` section and configure:

```toml
[dns]
upstreams = [
  "127.0.0.1#5335"
]
CNAMEdeepInspect = true
```

Restart services:

```bash
sudo systemctl restart unbound
sudo systemctl restart pihole-FTL
```

---

## 6. Test DNSSEC Validation

```bash
dig dnssec-failed.org @127.0.0.1 -p 5335     # should return SERVFAIL
dig sigok.verteiltesysteme.net @127.0.0.1 -p 5335  # should return NOERROR
dig +dnssec cloudflare.com @127.0.0.1 -p 5335      # NOERROR + “ad” flag
```

✅ Results:
- `dnssec-failed.org` → SERVFAIL  
- `sigok.verteiltesysteme.net` → NOERROR  
- `cloudflare.com` → AD flag confirmed (Authenticated Data)

---

## 7. Firewall Troubleshooting & Resolution – Allowing Port 53 and 80

Initially, Pi-hole was unreachable from LAN devices because **UFW’s default policy** was `deny (incoming)` and no rules existed for DNS or web traffic.

### Problem
- Windows `nslookup` timed out:  
  `DNS request timed out. Server: UnKnown (192.168.77.10)`

### Root Cause
UFW blocked DNS port 53 and HTTP/HTTPS (80/8443) by default.

### Solution

Allow DNS and web access for LAN:

```bash
sudo ufw allow from 192.168.77.0/24 to any port 53 proto tcp
sudo ufw allow from 192.168.77.0/24 to any port 53 proto udp
sudo ufw allow from 192.168.77.0/24 to any port 80
sudo ufw allow from 192.168.77.0/24 to any port 8443
sudo ufw reload
```

Verify rules:

```bash
sudo ufw status numbered
```
 Example final ruleset:

| Port | Protocol | Purpose | From | Status |
|------|-----------|----------|-------|--------|
| 53 | TCP/UDP | Pi-hole DNS | 192.168.77.0/24 | ✅ |
| 80 | TCP | HTTP web UI | 192.168.77.0/24 | ✅ |
| 8443 | TCP | HTTPS web UI | 192.168.77.0/24 | ✅ |
| 2256 | TCP | SSH (key‑based) | 192.168.77.0/24 | ✅ |
| Others | — | — | — | 🔒 Denied |

After these were added, `nslookup google.com 192.168.77.10` succeeded:

```
Server:  pi.hole
Address:  192.168.77.10
Non-authoritative answer:
Name:    google.com
Addresses: 142.250.176.14
```

This confirmed the Pi-hole DNS was reachable and functioning as intended.

---

## 8. Windows Client Configuration

Manually set your DNS server to Pi-hole:

**Preferred DNS:** `192.168.77.10`  
**Alternate DNS:** `192.168.77.1` (router, optional fallback)

Then flush and verify:

```powershell
ipconfig /flushdns
nslookup google.com
```

 Expected output:

```
Server:  pi.hole
Address: 192.168.77.10
Name:    google.com
Address: 142.250.xxx.xxx
```

---

## 9. Final State

| Component | Status | Description |
|------------|---------|-------------|
| **Pi-hole** | 🟢 Active | Handles LAN DNS and ad-blocking |
| **Unbound** | 🟢 Active | Performs recursive DNS lookups with DNSSEC |
| **Firewall** | 🟢 Hardened | LAN-only DNS, HTTPS, SSH |
| **DNSSEC** | 🟢 Verified | SERVFAIL for bad signatures, AD flag for valid |
| **Windows PC** | 🟢 Connected | Uses Pi-hole DNS successfully |

---

✅ **Phase 3 Complete**  
Your Raspberry Pi now provides **private, recursive, DNSSEC‑validated, network‑wide ad‑blocking**.

Next up → optional **Phase 4: Root Hints Auto‑Update & Performance Tuning.**