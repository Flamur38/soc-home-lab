# SPLUNK — SIEM Server

**Status:** Complete | **IP:** 192.168.30.10 | **Version:** Splunk Enterprise 10.2.2

## VM Configuration

| Setting | Value |
|---|---|
| OS | Ubuntu Server 24.04 LTS |
| RAM | 8GB |
| CPU | 2 cores |
| Storage | 80GB |
| Adapter | Host-only vboxnet2 |

## Network

- Management segment: 192.168.30.0/24
- Receives logs from DC01 and WIN11-CLIENT on port 9997 (Universal Forwarder)
- Receives pfSense syslog on UDP port 514

## What was configured

- Static IP via netplan (enp0s3)
- Splunk Enterprise installed via .deb package
- Boot-start enabled
- Port 9997 receiver enabled
- UDP 514 syslog input enabled
- Splunk Add-on for Microsoft Windows installed

## Log Sources

| Source | Method | Index |
|---|---|---|
| WIN11-CLIENT Security log | Universal Forwarder | main |
| WIN11-CLIENT Sysmon | Universal Forwarder | main |
| DC01 Security log | Universal Forwarder | main |
| DC01 Sysmon | Universal Forwarder | main |
| pfSense firewall | Syslog UDP 514 | main |

## Key Splunk Queries

```splunk
# Failed logins
index=main source="WinEventLog:Security" EventCode=4625

# Sysmon process events
index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"

# DC01 logs
index=main host="DC01" source="WinEventLog:Security"

# pfSense firewall logs
index=main sourcetype=syslog host="192.168.10.254"
```
