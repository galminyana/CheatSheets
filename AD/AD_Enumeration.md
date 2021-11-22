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

### Domain (or subdomain) Enumeration
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
### Groups Enumeration
---
#### Get All Groups in Current Domain
```powershell
C:> Get-DomainGroup | select Name
C:> Get-DomainGroup -Domain <targetdomain>
C:> Get-ADGroup -Filter * | select Name
C:> Get-ADGroup -Filter * -Properties *
```
#### Get all Groups Containing "admin" in the Group Name
```powershell
C:> Get-DomainGroup *admin*
C:> Get-ADGroup -Filter 'Name -like "*admin*"' | select Name
```
#### Get Members of Domain Admins Group
```powershell
C:> Get-DomainGroupMember -Identity "Domain Admins" -Recurse
C:> Get-ADGroupMember -Identity "Domain Admins" -Recursive
```
#### Get Group Membership of a User
```powershell
C:> Get-DomainGroup UserName "user"
C:> Get-ADPrincipalGroupMembership -Identity user
```
#### List Local Groups on Machine (Requires Admin Privs on non-dc Machines)
```powershell
C:> Get-NetLocalGroup -ComputerName computer -ListGroups
```
#### Get Members of Local Groups on a Machine (Requires Admin Privs on non-dc Machines)
```powershell
C:> Get-NetLocalGroup -ComputerName computer -Recurse
```
#### Get Members of Local Group "Administrators" on Machine (Requires Admin Privs on non-dc Machines)
```powershell
C:> Get-NetLocalGroupMember -ComputerName computer -GroupName Administrators
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
> `%{}` Iteration for the results before the pipe. `$_` Variabe for the iterated value

### Misc Enumeration
---
#### Active Users Logged on Computer (Requires Local Admin Rights)
```powershell
C:> Get-NetLoggedon -ComputerName servername
```
#### Locally Logged Users on Computer (Requires Remote Registry in the Target)
```powershell
C:> Get-LoggedonLocal -ComputerName computer
```
#### Get Last Logged User on Computer  (Requires Local Admin Rights and Remote Registry)
```powershell
C:> Get-LastLoggedOn -ComputerName servername
```
### Shares and Files Enum
---
#### Find Shares on Hosts in Current Domain
```powershell
C:> Invoke-ShareFinder -Verbose
```
#### Find Sensitive Files on Computers in the Domain
```powershell
C:> Invoke-FileFinder -Verbose
```
#### Get Fileservers on Domain
```powershell
C:> Get-NetFileServer
```
### GPO Enumeration
---
#### List of GPO in Current Domain
```powershell
C:> Get-DomainGPO
C:> Get-DomainGPO -ComputerIdentity computer
```
#### Get GPO who use Restricted Groups or groups.xml for Interesting Users
```powershell
C:> Get-DomainGPOLocalGroup
```
#### Users Which are in Local Group of a Machine using GPO
```powershell
C:> Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity computer
```
#### Get Machines where the Usert is Member of a Specific Group
```powershell
C:> Get-DomainGPOUserLocalGroupMapping -Identity user -Verbose
```
### OU Enumeration
---
#### Get Domain OUs
```powershell
C:> Get-DomainOU
C:> Get-ADOrganizationalUnit -Filter * -Properties *
```
#### Get GPO Applied to an OU
Read GPOname from gplink attribute from `Get-NetOU`
```powershell
C:> Get-DomainGPO -Identity "{AB306569-220D-43FF-B03B-83E8F4EF8081}"
```
### ACL Enumeration
---
#### ACL Associated to Specified Object
```powershell
C:> Get-DomainObjectAcl -SamAccountName user -ResolveGUIDs
```
#### Get ACL Associated with the Specified PRefix to Use for Search
```powershell
C:> Get-DomainObjectAcl -SearchBase "LDAP://CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs -Verbose
```
#### Enumerate ACL using AD Module. Without Resolving GUIDs
```powershell
C:> (Get-Acl 'AD:\CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local').Access
```
#### Search for interesting ACEs
```powershell
C:> Find-InterestingDomainAcl -ResolveGUIDs
```
#### Get ACL Associated With Specified Path
```powershell
Get-PathAcl -Path "\\dcorp-dc.dollarcorp.moneycorp.local\sysvol"
```







