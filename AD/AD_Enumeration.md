## AD Enumeration

c:> pwershell.exe -nop -exec bypass

No Admin PRivileges required:
```powershell
c:> whoami /priv
```
### Tools:
---

- [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

- Active Directory PowerShell Module:

### Domain (or subdomain) ENumeration
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
```powershell
c:> Get-NetDomain
c:> Get-ADDomain
```
#### Get Object of Another Domain
```powershell
Get-NetDomain -Domain domain.local
Get-ADDomain -Identity donain.local
```
#### Get Domain SID for Current Domain
```powershell
Get-DomainSID
(Get-ADDDomain).DomainSID
```
#### Get Domain Controllers for Current Domain
```powershell
Get-NetDomainController
Get-ADDomainController
```
#### Get Domain Controllers for Another Domain
```powershell
Get-NetDomainController -Domain domain.local
Get-ADDomainController -DomainName domain.local -Discover
```
#### Get Domain Policy for Current Domain
```powershell
Get-DomainPolicy
(Get-DomainPolicy)."system access"
                  ."Kerberos Policy"
```				  
[PASTE RESULTS AND DESCRIPTION]

#### Get Domain Policy for Another Domain
```powershell
(Get-DomainPolicy -domain domain.local)."system access"
```
### Domain Users Enumeration
---

#### List of Users in the Domain
```powershell
Get-NetUser
Get-NetUser -Username user
Get-ADUser -Filter * -Properties *
Get-ADUser -Identity user -Properties *
```
#### List of properties for users in the current domain
```powershell
Get-UserProperty
GetUserProperty -Properties pwdlastset
Get-ADUser -Filter * -Properties * | select -First 1 | Get-Member -MemberType *Property | select Name
Get-ADUser -Filter * .PRoperties * | select name,@{expression={[datetime]::fromFileTime($_.pwdlastset)}}
```
