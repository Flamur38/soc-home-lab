# SPLUNK — SIEM Server

## Overview

Ubuntu Server running Splunk Enterprise. Central log collection point for all lab VMs.
Status: Complete
IP: 192.168.30.10 | Segment: Management (vboxnet2) | Version: Splunk Enterprise 10.2.2

---

## VM Configuration

| Setting | Value |
| --- | --- |
| OS | Ubuntu Server 24.04.4 LTS |
| RAM | 8GB |
| CPU | 2 cores |
| Storage | 80GB |
| Adapter 1 | Host-only vboxnet2 (192.168.30.x) |

---

## Step 1 — Install Ubuntu Server

Booted from Ubuntu Server 24.04 ISO. Followed installer:
- Installation type: Ubuntu Server (not minimized)
- Network: no IP shown (expected — host-only, no DHCP)
- Storage: entire disk
- Username: flamur | Hostname: splunk
- OpenSSH server: installed during setup

---

## Step 2 — Set static IP

Interface identified as enp0s3 via `ip a`.

Edited netplan config using tee (nano had indentation issues with YAML):

```bash
sudo tee /etc/netplan/00-installer-config.yaml << 'YAML'
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.30.10/24
      nameservers:
        addresses:
          - 8.8.8.8
YAML
sudo chmod 600 /etc/netplan/00-installer-config.yaml
sudo netplan apply
```

Result: enp0s3 UP with inet 192.168.30.10/24 confirmed.

---

## Step 3 — Enable SSH

SSH was installed but inactive (socket-activated only).

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

Connected from Ubuntu host:

```bash
ssh flamur@192.168.30.10
```

All remaining steps performed via SSH with full copy-paste from host.

---

## Step 4 — Download and install Splunk Enterprise

Temporarily added NAT as Adapter 2 in VirtualBox for internet access.
Brought up NAT interface:

```bash
sudo ip link set enp0s8 up
sudo networkctl up enp0s8
```

Downloaded Splunk 10.2.2 .deb from splunk.com (free account required):

```bash
wget -O splunk-10.2.2-80b90d638de6-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/10.2.2/linux/splunk-10.2.2-80b90d638de6-linux-amd64.deb"
```

Installed:

```bash
sudo dpkg -i splunk-10.2.2-80b90d638de6-linux-amd64.deb
```

Started Splunk (--run-as-root required on Ubuntu 24.04):

```bash
sudo /opt/splunk/bin/splunk start --accept-license --run-as-root
```

Enabled boot-start:

```bash
sudo /opt/splunk/bin/splunk enable boot-start --run-as-root
```

NAT adapter removed after install. Splunk web UI accessible at http://192.168.30.10:8000.

---

## Step 5 — Enable port 9997 receiver

Accepts logs from Splunk Universal Forwarders on DC01 and WIN11-CLIENT.

```bash
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:YOUR_PASSWORD --run-as-root
```

Verified in web UI: Settings → Forwarding and receiving → Configure receiving → 9997 listed.

---

## Step 6 — Enable UDP 514 syslog input

For pfSense firewall logs.

Web UI: Settings → Data Inputs → UDP → New Local UDP
- Port: 514
- Source type: syslog
- Index: main

---

## Step 7 — Install Splunk Add-on for Microsoft Windows

Required for correct parsing of WinEventLog and Sysmon sourcetypes.

Web UI: Apps → Find More Apps → search "Splunk Add-on for Microsoft Windows" → Install → Restart Splunk.

---

## Log Sources

| Source | Method | Port | Index |
| --- | --- | --- | --- |
| WIN11-CLIENT Security | Universal Forwarder | 9997 | main |
| WIN11-CLIENT Sysmon | Universal Forwarder | 9997 | main |
| DC01 Security | Universal Forwarder | 9997 | main |
| DC01 Sysmon | Universal Forwarder | 9997 | main |
| pfSense firewall | Syslog UDP | 514 | main |

---

## Key Splunk Queries

```splunk
# All security events from WIN11-CLIENT
index=main source="WinEventLog:Security"

# Sysmon events
index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"

# DC01 security events
index=main host="DC01" source="WinEventLog:Security"

# pfSense firewall logs
index=main sourcetype=syslog host="192.168.10.254"

# Failed logins (Event ID 4625)
index=main source="WinEventLog:Security" EventCode=4625

# Process creation (Sysmon Event ID 1)
index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1
```
