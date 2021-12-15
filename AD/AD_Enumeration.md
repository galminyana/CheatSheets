## AD Enumeration

- [Tools](#tools)
- [Domain (or subdomain) Enumeration](#domain--or-subdomain--enumeration)
- [PowerView Enumeration](#powerview-enumeration)
  * [Get Current Domain](#get-current-domain)
  * [Get Other Domains](#get-other-domains)
  * [Get Domain SID](#get-domain-sid)
  * [Get Domain Policy for Current Domain](#get-domain-policy-for-current-domain)
  * [Get Domain Policy for Another Domain](#get-domain-policy-for-another-domain)
  * [Get Domain Controllers](#get-domain-controllers)
  * [Domain Users](#domain-users)
  * [Users Properties](#users-properties)
  * [Computers in Current Domain](#computers-in-current-domain)
  * [Group and Group Members](#group-and-group-members)
  * [Shares](#shares)
  * [GPO Enumeration](#gpo-enumeration)
  * [OU Enumeration](#ou-enumeration)
  * [ACL Enumeration](#acl-enumeration)
  * [Domain Trusts Enumeration](#domain-trusts-enumeration)
  * [User Hunting](#user-hunting)
  * [Processes Enumeration](#processes-enumeration)
  * [Misc](#misc)
- [AD Module Enumeration](#ad-module-enumeration)
  * [Get Current Domain](#get-current-domain-1)
  * [Get Other Domains](#get-other-domains-1)
  * [Get Domain SID](#get-domain-sid-1)
  * [Get Domain Controllers](#get-domain-controllers-1)
  * [Domain Users](#domain-users-1)
  * [Users Properties](#users-properties-1)
  * [Computers in Current Domain](#computers-in-current-domain-1)
  * [Group and Group Members](#group-and-group-members-1)
  * [OU Enumeration](#ou-enumeration-1)
  * [ACL](#acl)
- [BloodHound Enumeration](#bloodhound-enumeration)
  * [Setup:](#setup-)
  * [Run Collectors:](#run-collectors-)


### Tools
---
No Admin Privileges required:
```powershell
c:> whoami /priv
```

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
### PowerView Enumeration
---
#### Get Current Domain
```powershell
C:> Get-NetDomain
C:> Get-Domain
```
#### Get Other Domains
```powershell
C:> Get-NetDomain -Domain domain.local
```
#### Get Domain SID
```powershell
C:> Get-DomainSID
```
#### Get Domain Policy for Current Domain
```powershell
C:> Get-DomainPolicy
C:> (Get-DomainPolicy)."system access"
                      ."Kerberos Policy"
```	  
#### Get Domain Policy for Another Domain
```powershell
C:> (Get-DomainPolicy -domain domain.local)."system access"
```
#### Get Domain Controllers
```powershell
C:> Get-NetDomainController
C:> Get-NetDomainController -Domain domain.local
```
#### Domain Users
```powershell
C:> Get-NetUser
C:> Get-NetUser -Username <user>
C:> Get-DomainUser 
C:> Get-DomainUser -Identity <user>

# Get Users for Services
C:> Get-DomainUser -SPN

# Save results to File
C:> Get-DomainUser | Out-File -FilePath .\DomainUsers.txt
```
#### Users Properties 
```powershell
C:> Get-DomainUser -Identity <user> Properties *
C:> Get-UserProperty

# List specific properties
C:> Get-UserProperty -Properties pwdlastset
C:> Get-DomainUser -Properties samaccountname,logonCount | Format-List

#Enumerate user logged on a machine
C:> Get-NetLoggedon -ComputerName <computer>

#Enumerate Session Information for a machine
C:> Get-NetSession -ComputerName <computer>

# Search string in user attributes
C:> Get-DomainUser -LDAPFilter "Description=*built*" | select name,Description      
```
#### Computers in Current Domain
```powershell
C:> Get-DomainComputer | select Name
C:> Get-DomainComputer -OperatingSystem "*Server 2016*"            
C:> Get-DomainComputer -Ping                                       

C:> Get-DomainComputer -Properties OperatingSystem, Name, DnsHostName | Sort-Object -Property DnsHostName

#Enumerate Live machines 
C:> Get-DomainComputer -Ping -Properties OperatingSystem, Name, DnsHostName | Sort-Object -Property DnsHostName

# Active Users Logged on Computer (Requires Local Admin Rights)
C:> Get-NetLoggedon -ComputerName <computer>

# Domain computers of the specified domain where specific users are logged in
Find-DomainUserLocation -Domain <domain_name> | Select-Object UserName, SessionFromName

# Locally Logged Users on Computer (Requires Remote Registry in the Target)
C:> Get-LoggedonLocal -ComputerName <computer>

# Get Last Logged User on Computer  (Requires Local Admin Rights and Remote Registry)
C:> Get-LastLoggedOn -ComputerName <computer>
```
> `Get-DomainComputer` has a filter flag, `-SearchBase`, that filters by  the `distinguishedname`.

#### Group and Group Members
```powershell
# List Domain Groups
C:> Get-DomainGroup | select Name
C:> Get-DomainGroup -Domain <targetdomain>

# Groups containing "admin" in the Group Name
C:> Get-DomainGroup *admin*

# Group Membership for given user
C:> Get-DomainGroup -UserName <user>

# Domain members that belong to a given group
C:> Get-DomainGroup -Identity "<group>"
C:> Get-NetGroupMember -Identity "<group>"
C:> Get-DomainGroupMember -Identity "<group>" -Recurse

# Return members of Specific Group (eg. Domain Admins & Enterprise Admins)
C:> Get-DomainGroup -Identity <group> | Select-Object -ExpandProperty Member 
C:> Get-DomainGroupMember -Identity <group> | Select-Object MemberDistinguishedName

# List Local Groups on Machine (Requires Admin Privs on non-dc Machines)
C:> Get-NetLocalGroup 
C:> Get-NetLocalGroup -ComputerName <computer> -Recurse

# Enumerate members of a specific local group on the local (or remote) machine. Requires local admin rights on the remote machine
C:> Get-NetLocalGroupMember -GroupName Administrators | Select-Object MemberName, IsGroup, IsDomain
C:> Get-NetLocalGroupMember -ComputerName computer -GroupName Administrators
```
#### Shares
```powershell
# Enumerate Domain Shares
C:> Find-DomainShare

# Enumerate Domain Shares the current user has access
C:> Find-DomainShare -CheckShareAccess

# Enumerate "Interesting" Files on accessible shares
C:> Find-InterestingDomainShareFile -Include *passwords*

# Find Shares on Hosts in Current Domain
C:> Invoke-ShareFinder -Verbose

# Find Sensitive Files on Computers in the Domain
C:> Invoke-FileFinder -Verbose

# Get Fileservers on Domain
C:> Get-NetFileServer
```
#### GPO Enumeration
```powershell
# GPO List in the Current Domain
C:> Get-DomainGPO
C:> Get-DomainGPO -ComputerIdentity <computer>

C:> Get-DomainGPO -Properties DisplayName | Sort-Object -Property DisplayName

# Enumerate all GPOs to a specific computer
C:> Get-DomainGPO -ComputerIdentity <computer> -Properties DisplayName | Sort-Object -Property DisplayName

# Users that are part of a Machine's local Admin group
C:> Get-DomainGPOComputerLocalGroupMapping -ComputerName <computer>

# Get GPO who use Restricted Groups or groups.xml for Interesting Users
C:> Get-DomainGPOLocalGroup

# Users Which are in Local Group of a Machine using GPO
C:> Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity <computer>

## Get Machines where the Usert is Member of a Specific Group
C:> Get-DomainGPOUserLocalGroupMapping -Identity <user> -Verbose
```
#### OU Enumeration
```powershell
# Get Domain OUs
C:> Get-DomainOU

# Get GPO Applied to an OU
#    Read GPOname from gplink attribute from `Get-NetOU`
C:> Get-DomainGPO -Identity "{AB306569-220D-43FF-B03B-83E8F4EF8081}"

# List Computers in a OU. List the OU by the name, and then  get the computers searching on the distinguished name of the AD tree:
C:>  Get-DomainOU -Identity "StudentMachines"| %{Get-DomainComputer -SearchBase $_.distinguishedname}
```

#### ACL Enumeration
```powershell
# ACL Associated to Specified Object
C:> Get-DomainObjectAcl -SamAccountName user -ResolveGUIDs

# Get ACL Associated with the Specified PRefix to Use for Search
C:> Get-DomainObjectAcl -SearchBase "LDAP://CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs -Verbose

# Search for interesting ACEs
C:> Find-InterestingDomainAcl -ResolveGUIDs
C:> Find-InterestingDomainAcl -Identity <user or group> -ResolveGUIDs
# ACL for a username or group
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IDentityreferencename -match "<username or group>"}

# Get ACL Associated With Specified Path
C:> Get-PathAcl -Path "\\dcorp-dc.dollarcorp.moneycorp.local\sysvol"

# Use Invoke-ACLScanner: To get the ActiveDirectory Rights for a defined group on user:
C:> Invoke-ACLScanner -ResolveGUIDs | ?{$_.IdentityReferenceName -match "GROUPNAME"} | select Objectdn,identityreferencename,activedirectoryrights

# Check if Current User has GenericAll Rights on the AD Object for USER. Interestig to see the chance to change password for USER3
C:> Get-ObjectAcl -SamAccountName USER -ResolveGUIDs | ? {$_.ActiveDirectoryRights -eq "GenericAll"}  

# Check if USER can Force Password Change on USER2. Usefull to change user password
C:> Get-ObjectAcl -SamAccountName USER2 -ResolveGUIDs | ? {$_.IdentityReference -eq "DOMAIN\USER"}
###  If The USER2 has the `User-Force-Change-Password` in the ObjectType property for USER on it's ACL, password can be changed
     C:> Set-DomainUserPassword -Identity delegate -Verbose
     C:> runas /user:DOMAIN\USER2 cmd
     [Asks for a password and then spawns the shell] 
###  Also password can be changed like this
     C:> Set-DomainUserPassword -Identity USER2 -AccountPassword (ConvertTo-SecureString 'PASSWORD' -AsPlainText -Force) -Verbose
```
#### Domain Trusts Enumeration
```powershell
# Enumerate Domain Trust Relationships of the Current User
C:> Get-NetDomainTrust

# Domain Trust Mapping
C:> Get-DomainTrust
C:> Get-DomainTrust -Domain us.dollarcorp.moneycorp.local

# External Trust for Domain. Are those domains whose `TrustAttributes` equals to `"FILTER_SIDS"`
C:> Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | ?{$_.TrustAttributes -eq "FILTER_SIDS"}

# Get All Domains of the Forest for Current User
C:> Get-NetForestDomain
C:> Get-ForestDomain

# Forest Mapping
C:> Get-NetForestTrust

# Get All Global Catalogs for Current Forest
C:> Get-ForestGlobalCatalog
C:> Get-ForestGlobalCatalog -Forest eurocorp.local

# Map Trusts of a Forest
C:> Get-ForestTrust
C:> Get-ForestTrust -Forest eurocorp.local
```
#### User Hunting
```powershell
# Find machines on a domain or users on a given machine that are logged on
C:> Invoke-UserHunter -ComputerName COMPUTERNAME -Username *

# Find Machines Where Current User has Local Admin Access
C:> Find-LocalAdminAccess -Verbose
### This queries de DC for a list of computers using `Get-NetComputer` and then runs `Invoke-CheckLocalAdminAccess` on each machine.
###    Check the Scripts:
###    - `Find-WMILocalAdminAccess.ps1`
###    - `Find-PSRemotingLocalAdminAccess.ps1`: 
   C:> . .Find-PSRemotingLocalAdminAccess.ps1
   C:> Find-PSRemotingLocalAdminAccess <computer>
   C:> Enter-PSessiom -ComputerName computer-name
   [computer-name]: C:>

# Find Computers were Domain Admin (or specified user/group) has Sessions
C:> Find-DomainUserLocation -Verbose
C:> Find-DomainUserLocation -UserGroupIdentity "RDPUsers"

# Computers Where a Domain Admin Session is Available and Current User has Admin Access
C:> Find-DomainUserLocation -CheckAccess

# Find Computers Where a Domain Admin Session is Available (File Servers and Distributed File Servers)
C:> Find-DomainUserLocation -Stealth

# Test if user has access to computer
C:> Test-AdminAccess -ComputerName <computer>

# List domain Computers and user access
C:> Get-DomainComputer | Test-AdminAccess
```
#### Processes Enumeration
```powershell
# Get running processes for a given remote machine
C:> Get-NetProcess -ComputerName COMPUTERNAME -RemoteUserName DOMAIN\USER -RemotePassword PASSWORD | ft
```
#### Misc
```powershell
# Check if USER can add someone to Admins Group
### Get the Distinguished Name of Domain Admins Group
    C:> Get-NetGroup "domain admins" -FullData
### Check Permissions over that ACL
    C:> Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local" -and $_.IdentityReference -eq "DOMAIN\USER"}
```

### AD Module Enumeration
---
#### Get Current Domain
```powershell
C:> Get-ADDomain
```
#### Get Other Domains
```powershell
C:> Get-ADDomain -Identity domain.local
```
#### Get Domain SID
```powershell
C:> (Get-ADDDomain).DomainSID
```
#### Get Domain Controllers
```powershell
C:> Get-ADDomainController
C:> Get-ADDomainController -DomainName domain.local -Discover
```
#### Domain Users
```powershell
C:> Get-ADUser -Filter * -Properties *
C:> Get-ADUser -Identity <user> -Properties *
C:> Get-ADUser -Filter * -Identity <user> -Properties *
```
#### Users Properties 
```powershell
C:> Get-ADUser -Filter * -Properties * | select -First 1 | Get-Member -MemberType *Property | select Name
C:> Get-ADUser -Filter * .PRoperties * | select name,@{expression={[datetime]::fromFileTime($_.pwdlastset)}}

# Search in user properties
C:> Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | select name,Description
```
#### Computers in Current Domain
```powershell
C:> Get-ADComputer -Filter * | select Name
C:> Get-ADComputer -Filter * -Properties *
C:> Get-ADComputer -Filter * 'OperatingSystem -like "*Server 2016*"' -Properties OperatingSystem | select Name,OperatingSystem
C:> Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName} 
```

#### Group and Group Members
```powershell
C:> Get-ADGroup -Filter * | select Name
C:> Get-ADGroup -Filter * -Properties *

# Groups containing "Admin"
C:> Get-ADGroup -Filter 'Name -like "*admin*"' | select Name

# Get Members of Domain Admins Group
C:> Get-ADGroupMember -Identity "Domain Admins" -Recursive

# Get Group Membership for a User
C:> Get-ADPrincipalGroupMembership -Identity user
```
#### OU Enumeration
```powershell
# Get domain OU
C:> Get-ADOrganizationalUnit -Filter * -Properties *
```
- **Domain and Forests Trusts Enumeration**
```powershell
# Domain Trust Mapping
C:> Get-ADTrust
C:> Get-ADTrust -Identity us.dollarcorp.moneycorp.local

# Get Global Catalog for Current Forest
C:> Get-ADForest @ select -ExpandProperty GlobalCatalogs

# Map Trust of a Forest
C:> Get-ADTrust -Filter 'msDS-TrustForestTrustInfo -ne "$null"'

# Ennumerate Forest Trust
C:> Get-ADForest
C:> Get-ADForest -Identity <ForestName>

#Domains of Forest Enumeration
C:> (Get-ADForest).Domains
```
#### ACL
```powerview
# Enumerate ACL using AD Module. Without Resolving GUIDs
C:> (Get-Acl 'AD:\CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local').Access
```
### BloodHound Enumeration
---
#### Setup:
  - Install and run neo4j:
   ```powershell
   C:> neo4j.bat install-service
   C:> neo4j.bat start
   ```
  - Go to http://localhost:7474 and log using `neo4j` and `neo4j`.
  - Change the password for `neo4j` user to the desireed one
  - Run `BloodHound.exe` from `BloodHound-win32-x64`
  - Will ask for credentials: bolt://localhost:7687  -  neo4j  -  PASSWORD

#### Run Collectors:
  - Powershell:
   ```powershell
   C:> . .\SharpHound.ps1
   C:> Invoke-BloodHound -CollectionMethod All -Verbose
   ```
  - Exe Collector
   ```powershell
   C:> .\SharpHound.exe --CollectionMethod All --LdapUsername <UserName> --LdapPassword <Password> --domain <Domain> --domaincontroller <Domain Controller's Ip> --OutputDirectory <PathToFile>
    ```
