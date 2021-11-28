## [AD Enumeration](#ad-enumeration)
  * **[Tools:](#tools-)**
  * **[Domain (or subdomain) Enumeration](#domain--or-subdomain--enumeration)**
    + [Get Current Domain](#get-current-domain)
    + [Get Object of Another Domain](#get-object-of-another-domain)
    + [Get Domain SID for Current Domain](#get-domain-sid-for-current-domain)
    + [Get Domain Controllers for Current Domain](#get-domain-controllers-for-current-domain)
    + [Get Domain Controllers for Another Domain](#get-domain-controllers-for-another-domain)
    + [Get Domain Policy for Current Domain](#get-domain-policy-for-current-domain)
    + [Get Domain Policy for Another Domain](#get-domain-policy-for-another-domain)
  * **[Domain Users Enumeration](#domain-users-enumeration)**
    + [List of Users in the Domain](#list-of-users-in-the-domain)
    + [List of properties for users in the current domain](#list-of-properties-for-users-in-the-current-domain)
    + [Search Strings in Users Attributes](#search-strings-in-users-attributes)
  * **[Groups Enumeration](#groups-enumeration)**
    + [Get All Groups in Current Domain](#get-all-groups-in-current-domain)
    + [Get all Groups Containing "admin" in the Group Name](#get-all-groups-containing--admin--in-the-group-name)
    + [Domain Members That Belong to a Given Group](#domain-members-that-belong-to-a-given-group)
    + [Get Members of Domain Admins Group](#get-members-of-domain-admins-group)
    + [Get Group Membership of a User](#get-group-membership-of-a-user)
    + [List Local Groups on Machine (Requires Admin Privs on non-dc Machines)](#list-local-groups-on-machine--requires-admin-privs-on-non-dc-machines-)
    + [Get Members of Local Groups on a Machine (Requires Admin Privs on non-dc Machines)](#get-members-of-local-groups-on-a-machine--requires-admin-privs-on-non-dc-machines-)
    + [Get Members of Local Group "Administrators" on Machine (Requires Admin Privs on non-dc Machines)](#get-members-of-local-group--administrators--on-machine--requires-admin-privs-on-non-dc-machines-)
  * **[Computers Enumeration](#computers-enumeration)**
    + [List of Computers in Current Domain](#list-of-computers-in-current-domain)
  * **[Misc Enumeration](#misc-enumeration)**
    + [Active Users Logged on Computer (Requires Local Admin Rights)](#active-users-logged-on-computer--requires-local-admin-rights-)
    + [Locally Logged Users on Computer (Requires Remote Registry in the Target)](#locally-logged-users-on-computer--requires-remote-registry-in-the-target-)
    + [Get Last Logged User on Computer  (Requires Local Admin Rights and Remote Registry)](#get-last-logged-user-on-computer---requires-local-admin-rights-and-remote-registry-)
  * **[Shares and Files Enum](#shares-and-files-enum)**
    + [Find Shares on Hosts in Current Domain](#find-shares-on-hosts-in-current-domain)
    + [Find Sensitive Files on Computers in the Domain](#find-sensitive-files-on-computers-in-the-domain)
    + [Get Fileservers on Domain](#get-fileservers-on-domain)
  * **[GPO Enumeration](#gpo-enumeration)**
    + [List of GPO in Current Domain](#list-of-gpo-in-current-domain)
    + [Get GPO who use Restricted Groups or groups.xml for Interesting Users](#get-gpo-who-use-restricted-groups-or-groupsxml-for-interesting-users)
    + [Users Which are in Local Group of a Machine using GPO](#users-which-are-in-local-group-of-a-machine-using-gpo)
    + [Get Machines where the Usert is Member of a Specific Group](#get-machines-where-the-usert-is-member-of-a-specific-group)
  * **[OU Enumeration](#ou-enumeration)**
    + [Get Domain OUs](#get-domain-ous)
    + [Get GPO Applied to an OU](#get-gpo-applied-to-an-ou)
    + [List Computers in a OU](#list-computers-in-a-ou)
  * **[ACL Enumeration](#acl-enumeration)**
    + [ACL Associated to Specified Object](#acl-associated-to-specified-object)
    + [Get ACL Associated with the Specified PRefix to Use for Search](#get-acl-associated-with-the-specified-prefix-to-use-for-search)
    + [Enumerate ACL using AD Module. Without Resolving GUIDs](#enumerate-acl-using-ad-module-without-resolving-guids)
    + [Search for interesting ACEs](#search-for-interesting-aces)
    + [Get ACL Associated With Specified Path](#get-acl-associated-with-specified-path)
    + [Use Invoke-ACLScanner](#use-invoke-aclscanner)
    + [Check if Current User has GenericAll Rights on the AD Object for USER](#check-if-current-user-has-genericall-rights-on-the-ad-object-for-user)
    + [Check if USER can Force Password Change on USER2](#check-if-user-can-force-password-change-on-user2)
  * **[Domain Trusts Enumeration](#domain-trusts-enumeration)**
    + [Enumerate Domain Trust Relationships of the Current User](#enumerate-domain-trust-relationships-of-the-current-user)
    + [Domain Trust Mapping](#domain-trust-mapping)
    + [External Trust for Domain](#external-trust-for-domain)
    + [Get All Domains of the Forest for Current User](#get-all-domains-of-the-forest-for-current-user)
    + [Forest Mapping](#forest-mapping)
  * **[User Hunting](#user-hunting)**
    + [Find machines on a domain or users on a given machine that are logged on](#find-machines-on-a-domain-or-users-on-a-given-machine-that-are-logged-on)
    + [Find Machines Where Current User has Local Admin Access](#find-machines-where-current-user-has-local-admin-access)
    + [Find Computers were Domain Admin (or specified user/group) has Sessions](#find-computers-were-domain-admin--or-specified-user-group--has-sessions)
    + [Computers Where a Domain Admin Session is Available and Current User has Admin Access](#computers-where-a-domain-admin-session-is-available-and-current-user-has-admin-access)
    + [Find Computers Where a Domain Admin Session is Available (File Servers and Distributed File Servers)](#find-computers-where-a-domain-admin-session-is-available--file-servers-and-distributed-file-servers-)
  * **[Processes Enumeration](#processes-enumeration)**
    + [Get running processes for a given remote machine](#get-running-processes-for-a-given-remote-machine)
  * **[Misc](#misc)**
    + [Check if USER can add someone to Admins Group](#check-if-user-can-add-someone-to-admins-group)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

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
#### Domain Members That Belong to a Given Group
```powershell
C:> Get-NetGroupMember -GroupName "Domain Users"
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

> `Get-DomainComputer` has a filter flag, `-SearchBase`, that filters by  the `distinguishedname`:

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
#### List Computers in a OU
List the OU by the name, and then  get the computers searching on the distinguished name of the AD tree:
```powershell
C:>  Get-DomainOU -Identity "StudentMachines"| %{Get-DomainComputer -SearchBase $_.distinguishedname}
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
C:> Get-PathAcl -Path "\\dcorp-dc.dollarcorp.moneycorp.local\sysvol"
```
#### Use Invoke-ACLScanner
To get the ActiveDirectory Rights for a defined group on user:
```powershell
C:> Invoke-ACLScanner -ResolveGUIDs | ?{$_.IdentityReferenceName -match "GROUPNAME"} | select Objectdn,identityreferencename,activedirectoryrights
```
#### Check if Current User has GenericAll Rights on the AD Object for USER
Interestig to see the chance to change password for USER3
```powershell
C:> Get-ObjectAcl -SamAccountName USER -ResolveGUIDs | ? {$_.ActiveDirectoryRights -eq "GenericAll"}  
```
#### Check if USER can Force Password Change on USER2
Usefull to change user password
```powerview
C:> Get-ObjectAcl -SamAccountName USER2 -ResolveGUIDs | ? {$_.IdentityReference -eq "DOMAIN\USER"}
```
If The USER2 has the `User-Force-Change-Password` in the ObjectType property for USER on it's ACL, password can be changed
```powerview
C:> Set-DomainUserPassword -Identity delegate -Verbose
C:> runas /user:DOMAIN\USER2 cmd
[Asks for a password and then spawns the shell] 
```
Also password can be changed like this
```powerview
C:> Set-DomainUserPassword -Identity USER2 -AccountPassword (ConvertTo-SecureString 'PASSWORD' -AsPlainText -Force) -Verbose
```
### Domain Trusts Enumeration
---
#### Enumerate Domain Trust Relationships of the Current User
```powershell
C:> Get-NetDomainTrust
```
#### Domain Trust Mapping
```powershell
C:> Get-DomainTrust
C:> Get-DomainTrust -Domain us.dollarcorp.moneycorp.local
C:> Get-ADTrust
C:> Get-ADTrust -Identity us.dollarcorp.moneycorp.local
```
#### External Trust for Domain
Are those domains whose `TrustAttributes` equals to `"FILTER_SIDS"`
```powershell
C:> Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```
#### Get All Domains of the Forest for Current User
```powerview
C:> Get-NetForestDomain
```
#### Forest Mapping
```powershell
C:> Get-NetForestTrust
```

- Get All Global Catalogs for Current Forest
```powershell
C:> Get-ForestGlobalCatalog
C:> Get-ForestGlobalCatalog -Forest eurocorp.local
C:> Get-ADForest @ select -ExpandProperty GlobalCatalogs
```

- Map Trusts of a Forest
```powershell
C:> Get-ForestTrust
C:> Get-ForestTrust -Forest eurocorp.local
C:> Get-ADTrust -Filter 'msDS-TrustForestTrustInfo -ne "$null"'
```
### User Hunting
---
#### Find machines on a domain or users on a given machine that are logged on
```powershell
C:> Invoke-UserHunter -ComputerName COMPUTERNAME -Username *
```
#### Find Machines Where Current User has Local Admin Access
```powershell
C:> Find-LocalAdminAccess -Verbose
```
This queries de DC for a list of computers using `Get-NetComputer` and then runs `Invoke-CheckLocalAdminAccess` on each machine.
Check the Scripts:
- `Find-WMILocalAdminAccess.ps1`
- `Find-PSRemotingLocalAdminAccess.ps1`: 
```powershell
C:> . .Find-PSRemotingLocalAdminAccess.ps1
C:> Find-PSRemotingLocalAdminAccess 
computer-name
C:> Enter-PSessiom -ComputerName computer-name
[computer-name]: C:>
```
- Enter-PSession: Function from Powershell Core to log into a remote session as the current user.
#### Find Computers were Domain Admin (or specified user/group) has Sessions
```powershell
C:> Find-DomainUserLocation -Verbose
C:> Find-DomainUserLocation -UserGroupIdentity "RDPUsers"
```
This queries DC for the current or provided domain for members of the given group using `Get-DomainGroupMember`, then gets a list of computers with `Get-DomainComputer` and list sessions of logged on users using `Get-NetSession` or `Get-NetLoggedOn` from each machine.

#### Computers Where a Domain Admin Session is Available and Current User has Admin Access
```powrshell
C:> Find-DomainUserLocation -CheckAccess
```
#### Find Computers Where a Domain Admin Session is Available (File Servers and Distributed File Servers)
```powershell
C:> Find-DomainUserLocation -Stealth
```
### Processes Enumeration
---
#### Get running processes for a given remote machine
```powershell
C:> Get-NetProcess -ComputerName COMPUTERNAME -RemoteUserName DOMAIN\USER -RemotePassword PASSWORD | ft
```
### Misc
---
#### Check if USER can add someone to Admins Group
Get the Distinguished Name of Domain Admins Group
```powershell
C:> Get-NetGroup "domain admins" -FullData
```
Check Permissions over that ACL
```powershell
C:> Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local" -and $_.IdentityReference -eq "DOMAIN\USER"}
```
