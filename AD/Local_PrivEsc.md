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
In the three cmdlets, a `AbuseFunction` field is returned with the command to run to gain privileges


