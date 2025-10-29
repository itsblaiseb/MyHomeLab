# Cisco Catalyst WS-C2960XR-48LPD-I

**Date:** 2025-10-28  
**Device:** Cisco Catalyst C2960XR-48LPD-I  
**Purpose:** Erase all previous VLAN and password configurations to start clean.

---

## Situation
I have configured the switch before but I forgot the password so I need to reset it via console.

---

## Things needed
- Console access via USB-to-RJ45 cable  
- Terminal software (PuTTY, TeraTerm, or `screen`)  
- Baud rate: 115200  
- Data bits: 8  
- Parity: None  
- Stop bits: 1

---

## Steps Performed
1. Powered on the switch and console in to verify connectivity and communication with the switch.  
2. After verification that I am able to communicate with the switch I then powered it off.  
3. Held "MODE" button.  
4. Powered on while holding MODE for more than 30-60 seconds and let go after the SYST lights went green.  
5. After releasing the button I was able to get to `switch:` prompt in terminal.

---

## Initialize Flash and Delete Configs using the following commands
```bash
flash_init
dir flash:
delete flash:config.text
delete flash:vlan.dat
