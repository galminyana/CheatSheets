## Local PrivEsc

- [PowerUP](#powerup)
  * [Find Vulnerable Services](#find-vulnerable-services)
  * [Abuse Found Vulnerable Services](#abuse-found-vulnerable-services)
- [PrivEsc](#privesc)
  * [Find Vulnerable Services](#find-vulnerable-services-1)
- [PrivescCheck](#privesccheck)
- [WinPEAS](#winpeas)

### PowerUP
---
```powershell
C:> . .\PowerUp.ps1
```
#### Find Vulnerable Services

- Run All Checks
```powershell
C:> Invoke-AllChecks
```
- Get Services with Unquoted Paths and Spaces in Name
```powershell
C:> Get-ServiceUnquoted -vebose
```
- Get Services where Current User can Write to the Binary Path or Change Arguments to Binary
```powershell
C:> Get-ModifiableServiceFile
```
- Get Services that Current User can Modify Configuration
```powershell
C:> Get-ModifiableService
```
#### Abuse Found Vulnerable Services
In the three cmdlets, a `AbuseFunction` field is returned with the command to run to gain privileges. To make the user member of Local Administrators, can use:
```powershell
# Make <user> member of the Admins group. Just need to relog session and that's it.
C:> Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName '<domain>\<user>'
```

### PrivEsc
---
```powershell
C:> Invoke-PrivEsc
```
- Groups   : Groups of our interest when checking ACLs.
- Mode     : Mode to use - full or lhf (default).
- Extended : Switch enables lookups that may last for several minutes.

#### Find Vulnerable Services
- Run All Checks
```powershell
C:> Invoke-PrivEsc
```
- Full Check for Groups
```powershell
C:> Invoke-Privesc -Groups 'Users,Everyone,"Authenticated Users"' -Mode full -Extended
```
### PrivescCheck
---
Syntax:
```powerview
Invoke-PrivescCheck [[-Report] <string>] [[-Format] {TXT | HTML | CSV | XML}] [-Extended] [-Experimental] [-Force] [-Audit] [-Silent]  [<CommonParameters>]
```
Usage:
```powershell
C:> . PrivescCheck.ps1
C:> Invoke-PrivescCheck
```
  
### WinPEAS
---
WinPEAS is a binary to enumerate possible paths to escalate privileges locally
```powershell
        quiet                Do not print banner
        notcolor             Don't use ansi colors (all white)
        domain               Enumerate domain information
        systeminfo           Search system information
        userinfo             Search user information`
        processinfo          Search processes information
        servicesinfo         Search services information
        applicationsinfo     Search installed applications information
        networkinfo          Search network information
        windowscreds         Search windows credentials
        browserinfo          Search browser information
        filesinfo            Search files that can contains credentials
        eventsinfo           Display interesting events information
        wait                 Wait for user input between checks
        debug                Display debugging information - memory usage, method execution time
        log[=logfile]        Log all output to file defined as logfile, or to "out.txt" if not specified

        Additional checks (slower):
        -lolbas              Run additional LOLBAS check
        -linpeas=[url]       Run additional linpeas.sh check for default WSL distribution, optionally provide custom linpeas.sh URL
```

