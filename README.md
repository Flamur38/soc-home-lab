# SOC Home Lab — Flamur Xani

A practical home lab built to demonstrate real SOC analyst skills including 
network segmentation, Active Directory, SIEM configuration, attack simulation, 
and incident detection.

## Lab Architecture

| VM | OS | IP | Purpose |
|----|----|----|---------|
| pfSense | pfSense CE 2.8.1 | 192.168.10.254 | Virtual firewall |
| DC01 | Windows Server 2022 | 192.168.10.10 | Active Directory DC |
| WIN10-CLIENT | Windows 10 | 192.168.10.20 | Domain-joined endpoint |
| SPLUNK | Ubuntu Server 22.04 | 192.168.30.10 | SIEM |
| UBUNTU-SRV | Ubuntu Server 22.04 | 192.168.10.30 | DVWA web target |
| KALI | Kali Linux | 192.168.20.10 | Attacker machine |

## Network Segments

- **LAN (192.168.10.0/24)** — DC01, WIN10-CLIENT, UBUNTU-SRV
- **ATTACKER (192.168.20.0/24)** — Kali, isolated from LAN
- **MGMT (192.168.30.0/24)** — Splunk SIEM

All traffic routed through pfSense CE virtual firewall.

## Attack Scenarios Documented

| # | Scenario | Tool | Detection |
|---|----------|------|-----------|
| 01 | Network Reconnaissance | nmap | pfSense logs |
| 02 | Brute Force Login | Hydra | Event ID 4625 |
| 03 | Credential Dumping | Mimikatz | Sysmon Event ID 10 |
| 04 | Lateral Movement | PsExec | Event ID 4648, 4672 |
| 05 | Web Application Attack | sqlmap | Apache access logs |

## Tools Used

- pfSense CE — firewall and network segmentation
- Splunk Enterprise (free tier) — SIEM
- Sysmon (SwiftOnSecurity config) — endpoint telemetry
- Splunk Universal Forwarder — log shipping
- DVWA — vulnerable web application target

## Certifications

- BTL1 — Security Blue Team (April 2026)
- CompTIA Security+ (August 2025)
- OffSec OSCC (January 2025)
- CompTIA A+ (June 2024)

## Repository Structure

- `/lab-setup` — VM configs, network setup, tool installation
- `/investigations` — Attack/detection write-ups in SOC report format
- `/splunk-queries` — SPL searches used for detection
