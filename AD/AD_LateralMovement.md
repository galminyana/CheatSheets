## Lateral Movement on Active Directory 


### PowerShell Remoting
---
- Enable PowerShell Remoting: From [M$ Documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enable-psremoting?view=powershell-7.2), creates rules on computer firewall to allow remote connections. 
```powershell
C:> Enable-PSRemoting 
```
- Create Remote Session: [M$ Documented](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/new-pssession?view=powershell-7.2)
```powershell
# Create a Session in Local Computer
C:> New-PSSession

# Create Session on Remote Computer
C:> $remote_computer = New-PSSesion -ComputerName <computer>

# Create Session on Multiple Computers
C:> $s1, $s2, $s3 = New-PSSession -ComputerName Server01,Server02,Server03
```
- Entering into Existing PSSession
```powershell
# A session created previously with New-PSSession
C:> Enter-PSSession -Session $remote_computer

# A Completelly New Session to Computer
C:> Enter-PSSession -ComputerName <computer>
```
