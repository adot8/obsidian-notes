
Contrary to how it sounds, BOFHound is not a BOF.  It's a Python script that is capable of parsing the raw output of the ldapsearch BOF (and a few other tools).

## Query Methodology

The aim of the game is to use small, efficient queries to build up a picture of the environment over several days or weeks.  We could start off by enumerating some basic information about the domain, computers, users, groups, OUs, and GPOs.

```powershell
beacon> ldapsearch (|(objectClass=domain)(objectClass=organizationalUnit)(objectClass=groupPolicyContainer)) *,ntsecuritydescriptor

beacon> ldapsearch (|(samAccountType=805306368)(samAccountType=805306369)(samAccountType=268435456)) --attributes *,ntsecuritydescriptor
```

BOFHound is designed to parse the raw cobalt strike logs stored on the team server, so we first need to copy them across to our desktop.

```powershell
attacker@DESKTOP-FGSTPS7:/mnt/c/Users/Attacker/Desktop$ scp -r attacker@10.0.0.5:/opt/cobaltstrike/logs .
attacker@10.0.0.5's password: Passw0rd!
```

Then run `bofhound` against the copied log directory.

```
attacker@DESKTOP-FGSTPS7:/mnt/c/Users/Attacker/Desktop$ bofhound -i logs/
attacker@DESKTOP-FGSTPS7:/mnt/c/Users/Attacker/Desktop$ ls -l

-rwxrwxrwx 1 attacker attacker 16072 Mar 12 12:06 computers_20250312_120659.json
-rwxrwxrwx 1 attacker attacker  1803 Mar 12 12:06 domains_20250312_120659.json
-rwxrwxrwx 1 attacker attacker 13792 Mar 12 12:06 gpos_20250312_120659.json
-rwxrwxrwx 1 attacker attacker 34772 Mar 12 12:06 groups_20250312_120659.json
drwxrwxrwx 1 attacker attacker  4096 Mar 12 12:06 logs
-rwxrwxrwx 1 attacker attacker  5690 Mar 12 12:06 ous_20250312_120659.json
-rwxrwxrwx 1 attacker attacker 21889 Mar 12 12:06 users_20250312_120659.json
```

After logging into the BloodHound UI, navigate to **Administration > File Ingest** and upload the parsed JSON files.  You can then start querying the data once it's been processed.  For example, find all kerberoastable users with the cypher query:

```powershell
MATCH (n:User) WHERE n.hasspn=true RETURN n
```

> Every object that is only represented by a SID, or 'no name or id' means that we haven't collected any data on it yet.  You would then go back to ldapsearch and query for data on those objects.  You can target a specific object using `(objectsid=[SID]) --attributes *,ntsecuritydescriptor`.

### Restricted groups
One piece of data that can't be collected using LDAP alone, is restricted group data.  These are local group memberships that are defined and applied via GPOs, and without this information, BloodHound could be missing a lot of edges.  For example, it will show a _Server Admins_ GPO that is linked to a _Member Servers_ OU, of which there are several members.  However, LDAP cannot tell us anything about the actual policies that this GPO applies to the computers.

![[Pasted image 20250713224021.png]]

Selecting a computer such as _lon-ws-1_ and looking at its properties will show no local 
admins because they are currently unknown.

One way to figure this out is to browse to the [gPCFileSysPath](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-ada1/6eb770b7-0c89-4a3e-a41e-2807d46880d8) of the GPO in SYSVOL, and have a look at the actual policy files inside the container.  Different policies are applied using different file formats, such as `.pol`, `.xml`, and `.inf`.  It really is a mish-mash.

Restricted group data is defined in a file called _GptTmpl.inf_ inside the Machine's SecEdit folder, e.g. `\\contoso.com\SysVol\contoso.com\Policies\{7A04B429-D293-45E8-A7BA-D630415B3D6A}\Machine\Microsoft\Windows NT\SecEdit\GptTmpl.inf`.

```txt
[Unicode]
Unicode=yes
[Version]
signature="$CHICAGO$"
Revision=1
[Group Membership]
*S-1-5-21-3926355307-1661546229-813047887-1107__Memberof = *S-1-5-32-544
*S-1-5-21-3926355307-1661546229-813047887-1107__Members =
```

This is telling us that a domain group of SID `S-1-5-21-3926355307-1661546229-813047887-1107` is a member of a local group of SID `S-1-5-32-544`.  We can search for the domain group SID in BloodHound to find that it's a _Server Admins_ group.

![[Pasted image 20250713224204.png]]

And we can simply lookup the local SID in Microsoft's [documentation](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers), where _S-1-5-32-544_ is the built-in Administrators group.  This means that any member of the Server Admins group will be a local administrator on computers in the Member Servers OU.

These paths can be added to BloodHound manually using cypher query.

```powershell
MATCH (x:Computer{objectid:'S-1-5-21-3926355307-1661546229-813047887-1110'})
MATCH (y:Group{objectid:'S-1-5-21-3926355307-1661546229-813047887-1107'})
MERGE (y)-[:AdminTo]->(x)
```

Where:

- S-1-5-21-3926355307-1661546229-813047887-1110 is the SID for lon-ws-1.
    
- S-1-5-21-3926355307-1661546229-813047887-1107 is the SID for the Server Admins group.

### WMI Filters

WMI filters are an important consideration when analysing GPOs.  They allow for additional criteria to be set on GPO application through a specific WMI query.  For example, a GPO may apply to an entire Workstations OU, but a WMI filter can restrict its application to computers with a certain Windows version, specific computer names, and so on.  Understanding WMI filters is crucial because they can result in situations where a GPO seemingly applies to one or more computers based on its OU membership, yet they're actually excluded due to a filter.

> At the time of writing, BloodHound does not collect, display, or evaluate WMI filters when building its graphs.

WMI filters and GPOs operate on a one-to-many relationship.  A GPO can only have one WMI filter applied at a time, whereas a WMI filter can be applied to multiple GPOs.

A WMI filter is applied to a GPO by modifying the GPO's `gPCWQLFilter` attribute.

```powershell
beacon> ldapsearch (objectClass=groupPolicyContainer) --attributes displayname,gPCWQLFilter

displayName: AppLocker
gPCWQLFilter: [contoso.com;{E91C83FB-ADBF-49D5-9E93-0AD41E05F411};0]
```

The above tells us that a WMI filter with the GUID `{E91C83FB-ADBF-49D5-9E93-0AD41E05F411}` is being applied to the AppLocker GPO.  The WMI filters themselves are stored in the **CN=System,CN=WmiPolicy,CN=SOM** container, and are of class **msWMI-Som**.  The display name is stored in the **msWMI-Name** attribute and actual content of the filter in **msWMI-Parm2**.

```powershell
beacon> ldapsearch (objectClass=msWMI-Som) --attributes name,msWMI-Name,msWMI-Parm2 --dn "CN=SOM,CN=WMIPolicy,CN=System,DC=contoso,DC=com"

Binding to 10.10.120.1

[*] Distinguished name: CN=SOM,CN=WMIPolicy,CN=System,DC=contoso,DC=com
[*] targeting DC: \\lon-dc-1.contoso.com
[*] Filter: (objectClass=msWMI-Som)
[*] Scope of search value: 3
[*] Returning specific attribute(s): name,msWMI-Name,msWMI-Parm2

--------------------
name: {E91C83FB-ADBF-49D5-9E93-0AD41E05F411}
msWMI-Name: Windows 10+
msWMI-Parm2: 1;3;10;61;WQL;root\CIMv2;SELECT * from Win32_OperatingSystem WHERE Version like "10.%";
```

Where:

- **1;3;10** is a constant value, no idea what they're for 😅
    
- **61** is the length of the WMI query.
    
- **WQL** is the query language (WMI Query Language).
    
- **root\CIMv2** is the query namespace.
    
- **SELECT * from Win32_OperatingSystem WHERE Version like "10.%"**  
    is the query itself (61 characters).

> This means that if this GPO were to linked to an OU that contained computers running Windows 7, Vista, 8, etc, then it would not be applied to them because they wouldn't match the WMI query in the filter.