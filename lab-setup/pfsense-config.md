# pfSense CE Setup — Virtual Firewall

## Purpose
Virtual firewall providing network segmentation between LAN, ATTACKER, 
and WAN segments. All inter-segment traffic routes through pfSense.

## VM Specifications
| Setting | Value |
|--------|-------|
| OS | pfSense CE 2.8.1 |
| RAM | 1GB |
| Storage | 20GB |
| Hypervisor | VirtualBox 7.x |

## Network Interfaces
| Interface | Adapter | Network | IP |
|-----------|---------|---------|-----|
| WAN | le0 | NAT | DHCP (10.0.2.15) |
| LAN | le1 | vboxnet0 | 192.168.10.254/24 |
| ATTACKER | le2 | vboxnet1 | 192.168.20.254/24 |

## Installation Steps
1. Downloaded pfSense CE 2.8.1 ISO from pfsense.org
2. Created VirtualBox VM (BSD/FreeBSD 64-bit, 1GB RAM, 20GB disk)
3. Attached 3 network adapters: NAT, vboxnet0, vboxnet1
4. Booted ISO, ran installer with default options
5. Assigned interfaces: le0=WAN, le1=LAN, le2=ATTACKER
6. Set LAN IP to 192.168.10.254/24 via console option 2
7. Enabled DHCP on LAN: 192.168.10.100-200
8. Accessed web UI at http://192.168.10.254
9. Added ATTACKER interface (OPT1/le2) via Interfaces → Assignments
10. Set ATTACKER IP to 192.168.20.254/24

## Pending Configuration
- [ ] Firewall rules: allow ATTACKER → LAN, block LAN → ATTACKER
- [ ] DHCP on ATTACKER segment
- [ ] Syslog forwarding to Splunk (192.168.30.10:514)
- [ ] Change admin password

## Network Diagram
(add diagram here after all VMs are built)
