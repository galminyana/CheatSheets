## AD Enumeration

c:> pwershell.exe -nop -exec bypass

No Admin PRivileges required:

c:> whoami /priv

### Tools:
---

- [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

- Active Directory PowerShell Module:

### Get Domain (or subdomain) on where the user is Logged
---
```powershell
c:> $ADClass = [System.DirectoryServices.ActiveDirectory.Domain]
c:> $ADClass::GetCurrentDomain()

Forest                  : homelab.local
DomainControllers       : {w2019-DC.homelab.local}
Children                : {}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  :
PdcRoleOwner            : w2019-DC.homelab.local
RidRoleOwner            : w2019-DC.homelab.local
InfrastructureRoleOwner : w2019-DC.homelab.local
Name                    : homelab.local
```

#### Get Current Domain
Get-NetDomain
Get-ADDomain

#### Get Object of Another Domain
Get-NetDomain -Domain domain.local
Get-ADDomain -Identity donain.local

#### Get Domain SID for Current Domain
Get-DomainSID
(Get-ADDDomain).DomainSID

#### Get Domain Controllers for Current Domain
Get-NetDomainController
Get-ADDomainController

#### Get Domain Controllers for Another Domain
Get-NetDomainController -Domain domain.local
Get-ADDomainController -DomainName domain.local -Discover