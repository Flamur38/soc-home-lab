# Active Directory Setup — DC01

## VM Specifications
| Setting | Value |
|--------|-------|
| OS | Windows Server 2022 Standard Evaluation |
| RAM | 4GB |
| CPU | 2 cores |
| Storage | 60GB |
| IP | 192.168.10.10/24 |
| Gateway | 192.168.10.254 (pfSense) |
| DNS | 127.0.0.1 (self) |

## Domain
- Domain name: `flamurlab.local`
- DSRM password: stored securely offline

## Users Created
| Username | OU |
|----------|----|
| alice.martin | OU=LabUsers |
| bob.weber | OU=LabUsers |
| carol.schmidt | OU=LabUsers |
| david.mueller | OU=LabUsers |
| eva.braun | OU=LabUsers |

## Audit Policies Enabled
| Subcategory | Success | Failure |
|-------------|---------|---------|
| Logon | Yes | Yes |
| Account Lockout | Yes | Yes |
| Special Logon | Yes | Yes |
| Credential Validation | Yes | Yes |
| Process Creation | Yes | Yes |
| Sensitive Privilege Use | Yes | Yes |

## Sysmon
- Version: 15.20
- Config: SwiftOnSecurity sysmonconfig-export.xml
- Status: Running as service (SysmonDrv + Sysmon64)
