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
```
#### Get Tenant Name, Authentication, Brand Name, and Domain Name
```powershell
C> Get-AADIntLoginInformation -UserName [USER]@[DOMAIN].onmicrosoft.com
```
#### Get Tenant ID
```powershell
C> Get-AADIntTenantID -Domain [DOMAIN].onmicrosoft.com
```
#### Get Tenant Domains
```powershell
C> Get-AADIntTenantDomains -Domain [DOMAIN].onmicrosoft.com
```
#### Get All Info
```powershell
C> Get-AADIntReconAsOutsider -DomainName [DOMAIN].onmicrosoft.com
```
