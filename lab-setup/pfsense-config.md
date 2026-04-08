# pfSense CE — Setup Notes

## Overview
pfSense CE 2.8.1 acting as virtual firewall. Segments LAN, ATTACKER, and WAN.
Status: Complete
Web UI: http://192.168.10.254

---

## VM Configuration

| Setting | Value |
|---------|-------|
| OS | pfSense CE 2.8.1 |
| RAM | 1GB |
| Storage | 20GB |
| Adapter 1 | NAT → WAN (le0) |
| Adapter 2 | Host-only vboxnet0 → LAN (le1) |
| Adapter 3 | Host-only vboxnet1 → ATTACKER (le2) |

---

## Step 1 — Create host-only networks in VirtualBox

Go to File → Tools → Network Manager → Host-only Networks.
Create 3 networks and set IPs manually via the Adapter tab.

| Network | IP | Subnet |
|---------|----|--------|
| vboxnet0 | 192.168.10.1 | 255.255.255.0 |
| vboxnet1 | 192.168.20.1 | 255.255.255.0 |
| vboxnet2 | 192.168.30.1 | 255.255.255.0 |

> **Note:** CLI command `vboxmanage hostonlyif ipconfig` returned E_ACCESSDENIED.
> Fix: use the GUI Network Manager instead — it works without permission issues.

---

## Step 2 — Create the VM

New VM in VirtualBox:

- Type: BSD / FreeBSD 64-bit
- RAM: 1024MB | Storage: 20GB dynamically allocated
- Attach pfSense CE ISO to IDE controller
- Set all 3 network adapters before first boot

---

## Step 3 — Install pfSense

Boot from ISO and follow the installer:

1. Welcome screen → Install pfSense → OK
2. Accept default keymap
3. Auto partition → entire disk
4. Wait for install → reboot

Interface assignment on first boot:

| Interface | Adapter |
|-----------|---------|
| WAN | le0 |
| LAN | le1 |
| ATTACKER | le2 (added later via web UI) |

---

## Step 4 — Set LAN IP via console

Select option 2 (Set interface IP address) → select interface 2 (LAN).

| Prompt | Answer |
|--------|--------|
| Configure via DHCP? | n |
| New IPv4 address | 192.168.10.254 |
| Subnet bit count | 24 |
| Upstream gateway | Enter (none) |
| Configure IPv6? | n |
| IPv6 address | Enter (none) |
| Enable DHCP server? | y |
| DHCP start address | 192.168.10.100 |
| DHCP end address | 192.168.10.200 |
| Revert to HTTP? | y |

> **Warning:** First attempt used 192.168.10.1 which conflicted with vboxnet0
> on the Ubuntu host. Fix: change LAN IP to 192.168.10.254.

Result: LAN set to 192.168.10.254/24. DHCP running. Web UI at http://192.168.10.254

---

## Step 5 — Access web UI

Open browser on Ubuntu host → http://192.168.10.254

Default credentials:
Username: admin
Password: pfsense
> **Warning:** Change the default password immediately after first login.

---

## Step 6 — Add ATTACKER interface

Go to Interfaces → Assignments.
le2 listed under Available network ports → click Add → Save.

Then go to Interfaces → ATTACKER (OPT1):

| Setting | Value |
|---------|-------|
| Enable interface | Yes |
| Description | ATTACKER |
| IPv4 Configuration Type | Static IPv4 |
| IPv4 Address | 192.168.20.254 / 24 |

Click Save → Apply Changes.

Result: ATTACKER interface active at 192.168.20.254/24

---

## Network Summary

| Interface | Adapter | IP |
|-----------|---------|-----|
| WAN | le0 | 10.0.2.15 (DHCP via NAT) |
| LAN | le1 | 192.168.10.254/24 |
| ATTACKER | le2 | 192.168.20.254/24 |

---

## Still to configure

- [ ] Firewall rules: allow ATTACKER → LAN, block LAN → ATTACKER
- [ ] DHCP on ATTACKER segment
- [ ] Syslog forwarding to Splunk (192.168.30.10:514) — after Splunk is built
