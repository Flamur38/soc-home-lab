# SOC Home Lab — Flamur Xani

A practical home lab built to demonstrate real SOC analyst skills including
network segmentation, Active Directory, SIEM configuration, attack simulation,
and incident detection.

## VM Status
| VM | IP | Status |
|----|----|--------|
| pfSense | 192.168.10.254 | ✅ Complete |
| DC01 | 192.168.10.10 | ✅ Complete |
| WIN11-CLIENT | 192.168.10.20 | ✅ Complete |
| SPLUNK | 192.168.30.10 | ✅ Complete |
| UBUNTU-SRV | 192.168.10.30 | ✅ Complete |
| KALI | 192.168.20.10 | ⏳ Pending |

## Lab Architecture
| VM | OS | IP | Purpose |
|----|----|----|---------|
| pfSense | pfSense CE 2.8.1 | 192.168.10.254 | Virtual firewall |
| DC01 | Windows Server 2022 | 192.168.10.10 | Active Directory DC |
| WIN11-CLIENT | Windows 11 Enterprise | 192.168.10.20 | Domain-joined endpoint |
| SPLUNK | Ubuntu Server 24.04 | 192.168.30.10 | SIEM |
| UBUNTU-SRV | Ubuntu Server 24.04 | 192.168.10.30 | DVWA web target |
| KALI | Kali Linux | 192.168.20.10 | Attacker machine |

## Network Segments
- **LAN (192.168.10.0/24)** — DC01, WIN11-CLIENT, UBUNTU-SRV
- **ATTACKER (192.168.20.0/24)** — Kali, isolated from LAN
- **MGMT (192.168.30.0/24)** — Splunk SIEM

## Attack Scenarios
| # | Scenario | Tool | Detection |
|---|----------|------|-----------|
| 01 | Network Reconnaissance | nmap | pfSense logs |
| 02 | Brute Force Login | Hydra | Event ID 4625 |
| 03 | Credential Dumping | Mimikatz | Sysmon Event ID 10 |
| 04 | Lateral Movement | PsExec | Event ID 4648, 4672 |
| 05 | Web Application Attack | sqlmap | Apache access logs |

## Certifications
- BTL1 — Security Blue Team (April 2026)
- CompTIA Security+ (August 2025)
- OffSec OSCC (January 2025)
- CompTIA A+ (June 2024)
