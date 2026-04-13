# KALI — Attacker Machine

## Overview

Kali Linux VM used to run attack scenarios against the lab environment.
Status: Complete
IP: 192.168.20.10 | Segment: ATTACKER (vboxnet1)

---

## VM Configuration

| Setting | Value |
| --- | --- |
| OS | Kali Linux (latest) |
| RAM | 4GB |
| CPU | 2 cores |
| Storage | 40GB |
| Adapter 1 | Host-only vboxnet1 (192.168.20.x) |

---

## Step 1 — Install Kali Linux

Booted from Kali ISO. Selected Graphical Install.
- Hostname: kali
- Username: flamy
- Desktop: Xfce (default)
- Partitioning: guided, entire disk

---

## Step 2 — Set static IP

Kali uses NetworkManager (nmcli), not netplan.
First boot with NAT adapter to update and verify tools, then swapped to vboxnet1.

Deleted old connection profiles and created clean static config:

```bash
sudo nmcli con delete internet
sudo nmcli con add type ethernet con-name lab ifname eth0 \
  ipv4.addresses 192.168.20.10/24 \
  ipv4.method manual \
  ipv4.dns 192.168.10.10 \
  ipv4.gateway 192.168.20.254
sudo nmcli con up lab
```

Result: eth0 UP with inet 192.168.20.10/24. Gateway 192.168.20.254 (pfSense ATTACKER interface).

Fixed hostname resolution:

```bash
echo "127.0.0.1 kali" | sudo tee -a /etc/hosts
```

---

## Step 3 — Update and verify tools

Temporarily used NAT adapter for internet access:

```bash
sudo apt update && sudo apt upgrade -y
```

All attack tools pre-installed in Kali — confirmed with:

```bash
which nmap hydra msfconsole sqlmap burpsuite
```

| Tool | Path | Scenario |
| --- | --- | --- |
| nmap | /usr/bin/nmap | Reconnaissance |
| hydra | /usr/bin/hydra | Brute force |
| msfconsole | /usr/bin/msfconsole | Exploitation |
| sqlmap | /usr/bin/sqlmap | Web attacks |
| burpsuite | /usr/bin/burpsuite | Web attacks |

---

## Step 4 — Configure pfSense firewall rules

Added rule to allow ATTACKER segment to reach LAN segment.

pfSense web UI (http://192.168.10.254) → Firewall → Rules → ATTACKER → Add:

| Field | Value |
| --- | --- |
| Action | Pass |
| Interface | ATTACKER |
| Protocol | Any |
| Source | ATTACKER subnets |
| Destination | LAN subnets |
| Description | Allow Kali to reach LAN |

Rule applied successfully.

---

## Step 5 — Verify connectivity

DC01 responds to ping. WIN11-CLIENT and UBUNTU-SRV block ICMP (Windows Firewall / Docker)
but confirmed reachable via nmap -Pn:

```bash
ping -c 2 192.168.10.10       # DC01 — responds
nmap -Pn -p 445 192.168.10.20 # WIN11-CLIENT — host up
nmap -Pn -p 80 192.168.10.30  # UBUNTU-SRV — host up
```

All LAN targets reachable from Kali. Ready for attack scenarios.

---

## Attack Tools Reference

| Tool | Usage | Scenario |
| --- | --- | --- |
| nmap | `nmap -sV -A 192.168.10.0/24` | 01 — Recon |
| hydra | `hydra -L users.txt -P pass.txt rdp://192.168.10.20` | 02 — Brute force |
| msfconsole | `use post/windows/gather/credentials/credential_collector` | 03 — Cred dump |
| evil-winrm | `evil-winrm -i 192.168.10.20 -u user -p pass` | 04 — Lateral movement |
| sqlmap | `sqlmap -u "http://192.168.10.30/vulnerabilities/sqli/?id=1"` | 05 — Web attack |
