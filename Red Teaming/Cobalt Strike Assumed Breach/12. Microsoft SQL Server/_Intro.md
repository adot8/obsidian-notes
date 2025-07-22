
MS SQL (Structured Query Language) Server is Microsoft's proprietary relational database management system.

Popular tools to interact with MS SQL servers include [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL), [SQLRecon](https://github.com/skahwah/SQLRecon), [SQL-BOF](https://github.com/Tw1sm/SQL-BOF), [sqlcmd](https://github.com/microsoft/go-sqlcmd), [HeidiSQL](https://www.heidisql.com/), and [SSMS](https://learn.microsoft.com/en-us/ssms/download-sql-server-management-studio-ssms).  This chapter will exclusively demonstrate SQL-BOF, but you can use any tool you prefer.

### Enumeration
#### SPNs

One way to enumerate SQL server instances is to query for the SPNs that are used to support Kerberos authentication.  This will give you the hostname of the SQL servers and the service accounts that are used to run them