
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
```