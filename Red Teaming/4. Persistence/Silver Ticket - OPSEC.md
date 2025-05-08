For silver ticket attacks, we can forge the TGS (instead of a TGT) of a valid service account. Encrypted and Signed by the hash of the service account (Golden ticket is signed by hash of krbtgt) of the service running with that account

Services rarely check PAC (Privileged Attribute Certificate). Services will allow access only to the services themselves

Silver tickets are a lot more OPSEC safe due to the nature of the TGS. And it's safe to get the RC4 (NTLM) hash because it is a machine account which is still commonly used

> [!NOTE] **NOTE**
> This has reasonable persistence period (default 30 day rotation for computer accounts)
> 
> The most common service account is the **machine account** 
> 
> We can leverage the machine account hashes by forging a TGS for the machine which will let us impersonate any user we want like a DA. 

**AVOID TARGETING TO THE DC; MDI MAKES A FOCUS ON IT**

Forging a service ticket. The service and machine name are needed:
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:b8f18421f92b4ef069829b1dfd0e10e9 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```

> [!NOTE] Note
> Similar command can be used for any other service on a machine. 
> Which services? `HOST`, `RPCSS`, `CIFS` and many more.
> **YOU MUST USE CIFS IF YOU WANT TO ACCESS THE MACHINE VIA SMB** - **IT IS ALL SERVICE DEPENDANT**

EDR hash a problem when you run `klist` so we can use the loader and Rubeus instead
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args klist
```

```powershell
winrs -r:dcorp-dc.dollarcorp.moneycorp.local cmd
```

Just like the Golden ticket, `/ldap` option queries DC for information related to the user to create the final command


![[Pasted image 20250508062959.png]]