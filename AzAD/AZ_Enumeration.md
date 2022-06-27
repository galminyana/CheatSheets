## Az Tenant Discovery and Recon

#### Get if Tenant is in use, name and federation
---

Username can be whatever value, domain, the one we want to check. Returns a XML file with some info:

https://login.microsoftonline.com/getuserrealm.srf?login=[USERNAME@DOMAIN]&xml=1

Result Example:
```xml
<RealmInfo Success="true">
  <State>4</State>
  <UserState>1</UserState>
  <Login>USERNAME@microsoft.onmicrosoft.com</Login>
  <NameSpaceType>Managed</NameSpaceType>                              <<-- Managed
  <DomainName>microsoft.onmicrosoft.com</DomainName>
  <IsFederatedNS>false</IsFederatedNS>                                <<-- Not federated
  <FederationBrandName>Microsoft</FederationBrandName>                <<-- Federation Brand Name
  <CloudInstanceName>microsoftonline.com</CloudInstanceName>
  <CloudInstanceIssuerUri>urn:federation:MicrosoftOnline</CloudInstanceIssuerUri>
</RealmInfo>
```
Non existing Domain, returns something like this:
```xml
<RealmInfo Success="true">
  <State>4</State>
  <UserState>1</UserState>
  <Login>USERNAME@nadiequiereundominioaquio.onmicrosoft.com</Login>
  <NameSpaceType>Unknown</NameSpaceType>
</RealmInfo>
```
#### Get Tenant ID
---
Replace DOMAIN with the domain to recon:

https://login.microsoftonline.com/[DOMAIN]/.well-known/openid-configuration

Result Example:
```xml

```

### Using AADInternals Tool
---
[https://github.com/Gerenios/AADInternals](https://github.com/Gerenios/AADInternals)

```powershell
C> Import-Module AADInternals.psd1 -Verbose

#### Get Tenant Name, Authentication, Brand Name, and Domain Name
C> Get-AADIntLoginInformation -UserName [USER]@[DOMAIN].onmicrosoft.com

#### Get Tenant ID
C> Get-AADIntTenantID -Domain [DOMAIN].onmicrosoft.com

#### Get Tenant Domains
C> Get-AADIntTenantDomains -Domain [DOMAIN].onmicrosoft.com

#### Get All Info
C> Get-AADIntReconAsOutsider -DomainName [DOMAIN].onmicrosoft.com
```

### MicroBuster
---
```powershell
C> Import-Module .\MicroBurst.psm1   

#### Enumerate Subdomains
C> Invoke-EnumerateAzureSubDomains -Base microsoft

#### Enumerate Shared Blobs
C> Invoke-EnumerateAzureBlobs -Base microsoft
```

### AzureAD Module
---
```powershell
#### Get all App obejects registered with the current Tenant
C> Get-AzureADApplication -All $true

#### Get all details about an app
C> Get-AzureADApplication -ObjectId <id> | fl *

#### Get an app based on the display name
C> Get-AzureADApplication -All $true | ?{$_.DisplayName -match "name"}

### Show apps with an application password (password not shown)
C> Get-AzureADApplicationPAsswordCredential
```  
