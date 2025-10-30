# Cisco Catalyst 2960XR Switch Configuration

**Date:** 2025-10-28  
**Device:** Cisco Catalyst WS-C2960XR-48LPD-I  
**Hostname:** MainSwitch  
**Purpose:** Base configuration for home NAS network setup after factory reset.

---

## Environment Overview
| Device | IP Address | Purpose |
|--------|-------------|----------|
| Cisco RV320 | 192.168.1.1 | Default Gateway |
| Cisco 2960XR | 192.168.1.2 | LAN Switch (management) |
---

## ⚙️ Configuration Summary
**Configuration performed via console session:**
- Set hostname → `MainSwitch`
- Assigned management IP → `192.168.1.2 / 24`
- Default gateway → `192.168.1.1`
- Created local user and enabled SSH
- Enabled HTTPS/HTTP management
- Configured GigabitEthernet 2/0/2–10 as access ports
- Enabled `spanning-tree portfast`
- Set timezone → HST (-10)
- Added MOTD banner → `Unauthorized Access is Strictly Prohibited`

---

## Key Commands Used
```bash
hostname MainSwitch
no ip domain-lookup
interface vlan 1
 ip address 192.168.1.2 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.1.1
username admin privilege 15 secret <hidden>
enable secret <hidden>
ip domain-name billonlab.local
crypto key generate rsa modulus 2048
ip ssh version 2
line vty 0 15
 login local
 transport input ssh
exit
line console 0
 login local
 exec-timeout 10
exit
service password-encryption
logging buffered 4096
clock timezone HST -10 0
banner motd ^CUnauthorized Access is Strictly Prohibited^C
