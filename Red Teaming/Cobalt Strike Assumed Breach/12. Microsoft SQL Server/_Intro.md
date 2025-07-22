
MS SQL (Structured Query Language) Server is Microsoft's proprietary relational database management system.

Popular tools to interact with MS SQL servers include [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL), [SQLRecon](https://github.com/skahwah/SQLRecon), [SQL-BOF](https://github.com/Tw1sm/SQL-BOF), [sqlcmd](https://github.com/microsoft/go-sqlcmd), [HeidiSQL](https://www.heidisql.com/), and [SSMS](https://learn.microsoft.com/en-us/ssms/download-sql-server-management-studio-ssms).  This chapter will exclusively demonstrate SQL-BOF, but you can use any tool you prefer.

### Enumeration
#### SPNs

One way to enumerate SQL server instances is to query for the SPNs that are used to support Kerberos authentication.  This will give you the hostname of the SQL servers and the service accounts that are used to run them

```powershell
beacon> ldapsearch (&(samAccountType=805306368)(servicePrincipalName=MSSQLSvc*)) --attributes name,samAccountName,servicePrincipalName

Binding to 10.10.120.1

[*] Distinguished name: DC=contoso,DC=com
[*] targeting DC: \\lon-dc-1.contoso.com
[*] Filter: (&(samAccountType=805306368)(servicePrincipalName=MSSQLSvc*))
[*] Scope of search value: 3
[*] Returning specific attribute(s): name,samAccountName,servicePrincipalName

--------------------
name: MSSQL Service
sAMAccountName: mssql_svc
servicePrincipalName: MSSQLSvc/lon-db-1.contoso.com:1433, MSSQLSvc/lon-db-1.contoso.com

retreived 1 results total

```

#### Port scanner

If they haven't been configured to support Kerberos authentication, then scanning for open ports [known](https://learn.microsoft.com/en-us/sql/sql-server/install/configure-the-windows-firewall-to-allow-sql-server-access) to be used by MS SQL server may be a suitable alternative.

```powershell
beacon> portscan 10.10.120.0/23 1433 arp 1024

10.10.120.20:1433

Scanner module is complete
```

#### SQL server info

The information you can collect about a SQL instance depends on the role(s) you have on that instance.  If the _SQLBrowser_ service is running (1434/UDP), then you can collect very minimal information about the server name, instance, and version.  This can be collected without having any role on the SQL instance.

```powershell
beacon> sql-1434udp 10.10.120.20

SQL Server Connection Info:

ZServerName;LON-DB-1;InstanceName;MSSQLSERVER;IsClustered;No;Version;16.0.1000.6;tcp;1433;;
```

If a user does not have at least the [_public_ role](https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/server-level-roles), then no further interaction is viable.  With the public role, we can collect a bit more information such as the SQL server process's PID.

```powershell
beacon> sql-info lon-db-1

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Extracting SQL server information
 |--> ComputerName            : lon-db-1
 |--> DomainName              : CONTOSO
 |--> ServicePid              : 3364
 |--> ServiceName             : MSSQLSERVER
 |--> ServiceAccount          : CONTOSO\mssql_svc
 |--> AuthenticationMode      : Windows and SQL Server Authentication
 |--> ForcedEncryption        : 0
 |--> Clustered               : No
 |--> SqlServerVersionNumber  : 16.0.1000.6
 |--> SqlServerMajorVersion   : 2022
 |--> SqlServerEdition        : Standard Edition (64-bit)
 |--> SqlServerServicePack    : RTM
 |--> OsArchitecture          : X64
 |--> OsMachineType           : ServerNT
 |--> OsVersion               : Windows Server 2022 Standard
 |--> OsVersionNumber         : 2022
 |--> CurrentLogin            : CONTOSO\rsteel
 |--> IsSysAdmin              : True
 |--> ActiveSessions          : 1

[*] Disconnecting from server
```

You can query for information about the current user's rights on the SQL instance, which is useful for seeing all the roles and permissions that you have available.  Roles such as _serveradmin_, _securityadmin_, and _sysadmin_ are of particular interest.

```powershell
beacon> sql-whoami lon-db-1

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Determining user permissions on lon-db-1
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

### Querying
The public role will also grant permission to query the database instance, which is useful for data-hunting.

```powershell
beacon> sql-query lon-db-1 "SELECT @@SERVERNAME"

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Executing custom query on lon-db-1

 | 
---
lon-db-1 | 

[*] Disconnecting from server
```

SQL-BOF provides specific commands to list databases, tables, and rows of interest.

- `sql-databases` - list the available databases.
    
- `sql-tables` - list the tables in a given database.
    
- `sql-columns` - list the columns in a given table.
    
- `sql-search` - search for columns in the given database that contain a keyword.