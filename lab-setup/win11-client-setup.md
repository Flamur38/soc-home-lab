# WIN11-CLIENT — Domain-joined Endpoint

## Overview
Windows 11 Enterprise domain-joined endpoint simulating a real user workstation.
Status: Complete
IP: 192.168.10.20 | Domain: flamurlab.local

---

## VM Configuration
| Setting | Value |
|---------|-------|
| OS | Windows 11 Enterprise Evaluation |
| RAM | 8GB |
| CPU | 2 cores |
| Storage | 80GB |
| Adapter 1 | Host-only vboxnet0 |

---

## Step 1 — Install Windows 11
Boot from ISO. Select region and keyboard layout.
Network screen shows No Internet (expected) → click I don't have internet.
Set up local account when prompted.

Note: TPM 2.0 must be enabled in VirtualBox VM Settings → System before booting.

---

## Step 2 — Set static IP
```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.20 -PrefixLength 24 -DefaultGateway 192.168.10.254
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10
```
DNS must point to DC01 (192.168.10.10) for domain join to work.

---

## Step 3 — Join the domain
```powershell
Add-Computer -DomainName "flamurlab.local" -Credential flamurlab\Administrator -Restart
```
Verify after reboot:
```powershell
systeminfo | findstr /B /C:"Domain"
# Result: Domain: flamurlab.local
```

---

## Step 4 — Install VirtualBox Guest Additions
VirtualBox menu → Devices → Insert Guest Additions CD image
Run VBoxWindowsAdditions.exe → reboot.
Required for shared folder access.

---

## Step 5 — Install Sysmon
Add shared folder in VirtualBox: /home/flamy/Downloads → name: shared → auto-mount.

```powershell
New-Item -ItemType Directory -Path "C:\Sysmon"
Copy-Item "Z:\Sysmon.zip" "C:\Sysmon\"
Copy-Item "Z:\sysmonconfig-export.xml" "C:\Sysmon\"
Expand-Archive C:\Sysmon\Sysmon.zip -DestinationPath C:\Sysmon
C:\Sysmon\Sysmon64.exe -accepteula -i C:\Sysmon\sysmonconfig-export.xml
```
Result: Sysmon64 v15.20 installed. SysmonDrv started. Sysmon64 started.

---

## Step 6 — Install Splunk Universal Forwarder
Downloaded splunkforwarder.msi on Ubuntu host, transferred via shared folder.
Installed via GUI installer:
- Admin credentials set
- Deployment server: skipped
- Receiving indexer: 192.168.30.10:9997

Created inputs.conf:
```
[WinEventLog://Security]
disabled = 0
index = main

[WinEventLog://System]
disabled = 0
index = main

[monitor://C:\Windows\System32\winevt\Logs\Microsoft-Windows-Sysmon%4Operational.evtx]
disabled = 0
index = main
sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

```powershell
Restart-Service SplunkForwarder
```

---

## Still to configure
- [ ] Verify logs in Splunk: index=main source="WinEventLog:Security"
- [ ] Verify Sysmon in Splunk: index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
