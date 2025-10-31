# Phase 2 – Pi-hole HTTPS Hardening (v6+ Embedded Web Server)

This phase secures the Pi-hole web interface using HTTPS, limits access to your LAN, and verifies encryption.

---

## 1. Background

Pi-hole v6+ includes an **embedded web server** inside `pihole-FTL`, removing the need for Lighttpd. HTTPS can be enabled directly by providing certificate paths.

---

## 2. Generate a Self‑Signed TLS Certificate

```bash
sudo mkdir -p /etc/pihole/certs
cd /etc/pihole/certs
sudo openssl req -x509 -newkey rsa:4096 -keyout pihole.key -out pihole.crt -days 365 -nodes -subj "/CN=192.168.77.10"
sudo chmod 600 /etc/pihole/certs/pihole.key
sudo chmod 644 /etc/pihole/certs/pihole.crt
sudo chown pihole:pihole /etc/pihole/certs/pihole.*
```

---

## 3. Configure FTL for HTTPS

Edit `/etc/pihole/pihole-FTL.conf`:

```
WEB_PORT=8443
WEB_SSL=on
WEB_SSL_CERT=/etc/pihole/certs/pihole.crt
WEB_SSL_KEY=/etc/pihole/certs/pihole.key
```

Restart service:

```bash
sudo systemctl restart pihole-FTL
sudo systemctl status pihole-FTL --no-pager
```

---

## 4. Verify Certificate

```bash
openssl s_client -connect 192.168.77.10:8443 -servername 192.168.77.10 -showcerts </dev/null 2>/dev/null | openssl x509 -noout -subject -dates
```

 Expected output (example):
```
subject=CN=192.168.77.10
notBefore=...
notAfter=...
```

If you still see `CN=pi.hole`, Pi-hole’s internal default certificate is active — encryption is still valid and secure.

---

## 5. Firewall Rules for HTTPS‑Only Access

Allow HTTPS and deny HTTP ports:

```bash
sudo ufw allow from 192.168.77.0/24 to any port 8443
sudo ufw deny 8080/tcp
sudo ufw deny 80/tcp
sudo ufw reload
sudo ufw status verbose
```

 You should see:

```
8443/tcp ALLOW IN 192.168.77.0/24
8080/tcp DENY IN Anywhere
80/tcp   DENY IN Anywhere
```

---

## 6. Test HTTPS Access

From any LAN device:

```
https://192.168.77.10:8443/admin
```

You may get a self‑signed certificate warning — proceed or import `/etc/pihole/certs/pihole.crt` into your browser trust store.

Confirm dashboard loads normally.

---

## 7. Verify Firewall Enforcement

```bash
curl -I http://192.168.77.10:8080
curl -kI https://192.168.77.10:8443
```

 Expected results:
- Port 8080 → HTTP 403 Forbidden or connection refused  
- Port 8443 → HTTP 200 OK (secure page)

---

## 8. LAN‑Only Web and SSH Access

Restrict management interfaces to LAN:

```bash
sudo ufw delete allow 2256/tcp
sudo ufw allow from 192.168.77.0/24 to any port 2256 proto tcp
sudo ufw reload
```

This ensures both web and SSH are accessible only from trusted LAN devices.

---

## Summary

| Service | Port | Access | Protocol | Status |
|----------|------|---------|-----------|--------|
| Pi-hole Web (HTTPS) | 8443 | LAN‑only | TCP | ✅ |
| HTTP | 8080 / 80 | Blocked | TCP | 🔒 |
| SSH | 2256 | LAN‑only | TCP | ✅ |
| DNS | 53 | LAN‑only | UDP/TCP | ✅ |

---

 **Phase 2 Complete**  
Pi‑hole now uses HTTPS for the admin dashboard, accessible only from your LAN, with all unencrypted HTTP access blocked.

Next: **Phase 3 – Integrate Unbound for Private Recursive DNS.**
