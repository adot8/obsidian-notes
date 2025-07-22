
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