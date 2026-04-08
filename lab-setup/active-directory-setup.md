# DC01 — Active Directory Domain Controller

## Overview
Windows Server 2022 promoted to Domain Controller for flamurlab.local.
Status: Complete
IP: 192.168.10.10 | DNS: 127.0.0.1 (self) | Domain: flamurlab.local

---

## VM Configuration

| Setting | Value |
|---------|-------|
| OS | Windows Server 2022 Standard Evaluation (Desktop Experience) |
| RAM | 4GB |
| CPU | 2 cores |
| Storage | 60GB |
| Adapter 1 | Host-only vboxnet0 |
| Download | microsoft.com/evalcenter (180 day free trial) |

---

## Step 1 — Install Windows Server 2022

Boot from ISO:

1. Select Windows Server 2022 Standard Evaluation (Desktop Experience)
2. Installation type → Custom
3. Select unallocated disk → Next
4. Wait for install and automatic reboot
5. Set Administrator password when prompted

---

## Step 2 — Set static IP

Open PowerShell as Administrator:

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.10 -PrefixLength 24 -DefaultGateway 192.168.10.254
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1
```

Result: Static IP 192.168.10.10/24. Gateway: 192.168.10.254. DNS: self.

---

## Step 3 — Rename computer and reboot

```powershell
Rename-Computer -NewName "DC01" -Restart
```

---

## Step 4 — Install AD DS and promote to Domain Controller

After reboot, open PowerShell as Administrator. Run in sequence:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

```powershell
Install-ADDSForest -DomainName "flamurlab.local" -InstallDns
```

When prompted:
- SafeModeAdministratorPassword → set strong password, store offline
- Confirm warnings → type A (Yes to All)

Result: Server reboots automatically. Domain: flamurlab.local.

---

## Step 5 — Create lab users

After reboot, open PowerShell as Administrator. Create OU first:

```powershell
New-ADOrganizationalUnit -Name "LabUsers" -Path "DC=flamurlab,DC=local"
```

Create 5 lab users:

```powershell
$users = @("alice.martin","bob.weber","carol.schmidt","david.mueller","eva.braun")
foreach ($u in $users) {
    New-ADUser -Name $u -SamAccountName $u -UserPrincipalName "$u@flamurlab.local" `
    -Path "OU=LabUsers,DC=flamurlab,DC=local" `
    -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
    -Enabled $true
}
```

Result: 5 users created in OU=LabUsers with no errors.

---

## Step 6 — Enable audit policies

> **Warning:** Run in Command Prompt as Administrator — not PowerShell.
> Run one line at a time. Each should return "The command was successfully executed."
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable
auditpol /set /subcategory:"Special Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
auditpol /set /subcategory:"Sensitive Privilege Use" /success:enable /failure:enable
> **Note:** Subcategory "Account Logon" does not exist on Windows Server 2022.
> Use "Credential Validation" instead.
> To list all valid names: `auditpol /list /subcategory:*`

Result: All 6 audit policies set successfully.

---

## Step 7 — Install Sysmon

> **Warning:** DC01 has no internet access (host-only network).
> Files must be transferred from the Ubuntu host via VirtualBox shared folder.
> VirtualBox Guest Additions must be installed on DC01 for shared folders to work.

On Ubuntu host, download both files:

```bash
wget https://download.sysinternals.com/files/Sysmon.zip
wget https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml
```

In VirtualBox → DC01 Settings → Shared Folders:

| Setting | Value |
|---------|-------|
| Folder path | /home/flamy/Downloads |
| Folder name | shared |
| Auto-mount | Yes |
| Access | Full |

In DC01 PowerShell, copy, extract and install:

```powershell
Copy-Item "Z:\Sysmon.zip" "C:\Sysmon\"
Copy-Item "Z:\sysmonconfig-export.xml" "C:\Sysmon\"
Expand-Archive C:\Sysmon\Sysmon.zip -DestinationPath C:\Sysmon
C:\Sysmon\Sysmon64.exe -accepteula -i C:\Sysmon\sysmonconfig-export.xml
```

Result: Sysmon64 v15.20 installed. SysmonDrv started. Sysmon64 started.

---

## Still to configure

- [ ] Install Splunk Universal Forwarder
- [ ] Configure inputs.conf → forward Security log + Sysmon to 192.168.30.10:9997
