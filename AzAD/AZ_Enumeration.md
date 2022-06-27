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
#### Connect to Azure
C> $password = ConvertTo-SecureString 'PASSWORD' -AsPlainText -Force
c> $creds = New-Object System.Management.Automation.PSCredential('USER@DOMAIN.onmicrosoft.com', $password)
c> Connect-AzureAD -Credential $creds

#### Enumerate Users
C> Get-AzureADUser -All $true

#### Enumerate Groups
C> Get-AzureADGroup -All $true

#### Get all App obejects registered with the current Tenant
C> Get-AzureADApplication -All $true

#### Get all details about an app
C> Get-AzureADApplication -ObjectId <id> | fl *

#### Get an app based on the display name
C> Get-AzureADApplication -All $true | ?{$_.DisplayName -match "name"}

#### Show apps with an application password (password not shown)
C> Get-AzureADApplicationPAsswordCredential

#### List Dinamic Groups
C> Get-AzureADMSGroup | ?{$_.GroupTypes -eq 'DynamicMembership'}

#### Invite USer to the Tenant
C>  New-AzureADMSInvitation -InvitedUserDisplayName "HackerInvited69" -InvitedUserEmailAddress "USERDOMAIN.onmicrosoft.com" -InviteRedirectUrl https://portal.azure.com -SendInvitationMessage $true
#   Then with the Invite Redeem URL returned by the CMDlet execution, go there and login as the user invited to accept
```  

### Az Module
---
```powershell
#### Connect to Az Tenant
C> $password = ConvertTo-SecureString 'PASSWORD' -AsPlainText -Force
C> $creds = New-Object System.Management.Automation.PSCredential('USER@DOMAIN.onmicrosoft.com', $password)
C> Connect-AzAccount -Credential $creds

### List resources where USEr has access
C> Get-AzResource

# If the user does not have access, the following exception arises
#    Get-AzResource : 'this.Client.SubscriptionId' cannot be null.
#    En línea: 1 Carácter: 1
#    + Get-AzResource
#    + ~~~~~~~~~~~~~~
#        + CategoryInfo          : CloseError: (:) [Get-AzResource], ValidationException
#        + FullyQualifiedErrorId : Microsoft.Azure.Commands.ResourceManager.Cmdlets.Implementation.GetAzureResourceCmdlet

### 

```
### AzureADPreview
---
Some CMDlets dont work properly and then need to use this Module.-
```powershell
#### Import Module
C> Import-Module .\AzureADPreview\2.0.2.149\AzureADPreview.psd1

#### Get Membership rule for Dynamic Group
C> Get-AzureADMSGroup | ?{$_.GroupTypes -eq 'DynamicMembership'} |select MembershipRule

```
