## AD Enumeration
No Admin Privileges required:
```powershell
c:> whoami /priv
```
### Tools:
---

- [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

```powershell
C:> Import-Module PowerView.ps1
C:> . PowerView.ps1
```

- [Active Directory PowerShell Module](https://docs.microsoft.com/en-us/powershell/module/addsadministration/?view=win10-ps):

```powershell
C:> Import-Module C:\AD Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
C:> Import-Module C:\AD Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```
- [BloodHound](https://github.com/BloodHoundAD/BloodHound)

### Domain (or subdomain) ENumeration
---
```powershell
C:> $ADClass = [System.DirectoryServices.ActiveDirectory.Domain]
C:> $ADClass::GetCurrentDomain()

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
C:> Get-NetDomain
C:> Get-ADDomain
```
#### Get Object of Another Domain
```powershell
C:> Get-NetDomain -Domain domain.local
C:> Get-ADDomain -Identity donain.local
```
#### Get Domain SID for Current Domain
```powershell
C:> Get-DomainSID
C:> (Get-ADDDomain).DomainSID
```
#### Get Domain Controllers for Current Domain
```powershell
C:> Get-NetDomainController
C:> Get-ADDomainController
```
#### Get Domain Controllers for Another Domain
```powershell
C:> Get-NetDomainController -Domain domain.local
C:> Get-ADDomainController -DomainName domain.local -Discover
```
#### Get Domain Policy for Current Domain
```powershell
C:> Get-DomainPolicy
C:> (Get-DomainPolicy)."system access"
                      ."Kerberos Policy"
```				  
[PASTE RESULTS AND DESCRIPTION]

#### Get Domain Policy for Another Domain
```powershell
C:> (Get-DomainPolicy -domain domain.local)."system access"
```
### Domain Users Enumeration
---

#### List of Users in the Domain
```powershell
C:> Get-NetUser
C:> Get-NetUser -Username user
C:> Get-DomainUser 
C:> Get-DomainUser -Identity user
C:> Get-ADUser -Filter * -Properties *
C:> Get-ADUser -Identity user -Properties *
```
#### List of properties for users in the current domain
```powershell
C:> Get-DomainUser -Identity user Properties *
C:> Get-DomainUser -Properties samaccountname,logonCount
C:> Get-UserProperty
C:> Get-UserProperty -Properties pwdlastset
C:> Get-ADUser -Filter * -Properties * | select -First 1 | Get-Member -MemberType *Property | select Name
C:> Get-ADUser -Filter * .PRoperties * | select name,@{expression={[datetime]::fromFileTime($_.pwdlastset)}}
```
#### Search Strings in Users Attributes
```powershell
C:> Get-DomainUser -LDAPFilter "Description=*built*" | select name Description      # Returns Users with the *built* text in description
C:> Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | select name,Description
```

### Computers Enumeration
---
#### List of Computers in Current Domain
```powershell
C:> Get-DomainComputer | select Name
C:> Get-DomainComputer -OperatingSystem "*Server 2016*"             # Computers whose OS is "Server 2016"
C:> Get-DomainComputer -Ping                                        # Computers Online from the Domain
C:> Get-ADComputer -Filter * | select Name
C:> Get-ADComputer -Filter * -Properties *
C:> Get-ADComputer -Filter * 'OperatingSystem -like "*Server 2016*"' -Properties OperatingSystem | select Name,OperatingSystem
C:> Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}  
```
| `%{}` Iteration for the results before the pipe. `$_` Variabe for the iterated value

