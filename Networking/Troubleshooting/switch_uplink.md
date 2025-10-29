# Uplink Connectivity Issue – Cisco 2960XR

**Date:** 2025-10-29  
**Device:** Cisco Catalyst WS-C2960XR-48LPD-I  
**Hostname:** MainSwitch  
**Related Devices:** Cisco RV320 Router (LAN2)  

---

## Problem Summary
After completing the base configuration for the switch, the switch was connected to the Cisco RV320 router (LAN2 port).  
However, the switch was not reachable over the network — ping to `192.168.1.2` failed with the message:
Reply from 192.168.1.100: Destination host unreachable


---

## Initial Observations
- Switch management VLAN (`Vlan1`) was **up/up** with IP `192.168.1.2`.  
- Uplink port `GigabitEthernet2/0/1` showed **up/up** but RV320 reported **10 Mbps** link speed.  
- VLAN configuration on RV320 showed VLANs disabled (default untagged VLAN 1).  
- All other devices on LAN were functional.  

---

## Troubleshooting Steps
1. **Verified switch interfaces**
   ```bash
   show ip interface brief
Vlan1 → up/up
Gi2/0/1 → up/up

2. Confirmed router port
Router LAN2 link = up
Reported 10 Mbps link speed (expected 1000 Mbps).

3. Attempted manual speed/duplex setting
   ```bash
    interface gi2/0/1
    speed 1000
    duplex full

Result: port went down (router couldn’t negotiate).

4. Reverted to auto
   ```bash
    interface gi2/0/1
    speed auto
    duplex auto
    no shutdown

5. Still no ping, so I moved the cable from Gi2/0/1 to Gi2/0/3.

6. After move:
Link negotiated at 1000 Mbps full duplex.
Ping to 192.168.1.2 succeeded.
SSH access confirmed.

### Possible Root Cause
During the factory reset and password recovery process, the switch was rebooted while the uplink cable on interface Gi2/0/1 remained connected to the RV320 router.  
This likely caused an incomplete link negotiation at boot, resulting in a degraded 10 Mbps connection and no Layer 2 communication.  
After moving the uplink to Gi2/0/3 (forcing a new negotiation), the issue resolved immediately.
