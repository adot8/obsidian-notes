MS SQL servers are generally deployed in plenty in a Windows domain

SQL Servers provide very good options for lateral movement as domain users can be mapped to database roles.

#### [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL)

###### Discovery (SPN Scanning) - find all mssql SPNs (default for mssql servers)
```powershell
Get-SQLInstanceDomain
```

###### Check Accessibility 
```powershell
Get-SQLConnectionTestThreaded

Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded -Verbose
```

###### Gather Information
```powershell
Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose
```

>**Note:** Review `ServiceName`, `AuthenticationMode`, `IsSysadmin` and `ActiveSessions`
#### Database Links
A database link allows a SQL Server to access external data sources like other SQL Servers and OLE DB data sources. In case of database links between SQL servers, that is, linked SQL servers it is possible to execute stored procedures

> **Note**: Database links work even across forest trusts.

###### Search for Database Links
```powershell
Get-SQLServerLink -Instance dcorp-mssql -Verbose
```
 
 Manual
```sql
select * from master..sysservers
```

###### Enumerating Database Links 
```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Verbose
```

Manual - The `Openquery()` function can be used to run queries on a linked database
```sql
select * from openquery("dcorp-sql1",'select * from master..sysservers')
```