#### Enumeration
```bash
ssh-audit $ip
```
##### Bruteforce
```bash
hydra -vV -l user -P ~/rockyou.txt $ip -I -t 10 ssh
```
##### Password protected SSH key
```bash
ssh2john id_rsa > ssh.hash

john --wordlist=./ssh.pass --rules=sshRules ssh.hash
```
##### Private key types
```bash
id_rsa
id_ecdsa
id_ecdsa_sk
id_ed25519
id_ed25519_sk
id_dsa
```

[Secure Shell](https://en.wikipedia.org/wiki/Secure_Shell) (`SSH`) enables two computers to establish an encrypted and direct connection within a possibly insecure network on the standard port `TCP 22`. This is necessary to prevent third parties from intercepting the data stream and thus intercepting sensitive data. The SSH server can also be configured to only allow connections from specific clients. An advantage of SSH is that the protocol runs on all common operating systems.

There are two competing protocols: `SSH-1` and `SSH-2`.
`SSH-2`, also known as SSH version 2, is a more advanced protocol than SSH version 1 in encryption, speed, stability, and security. For example, `SSH-1` is vulnerable to `MITM` attacks, whereas SSH-2 is not.

OpenSSH has six different authentication methods:

1. Password authentication
2. Public-key authentication
3. Host-based authentication
4. Keyboard authentication
5. Challenge-response authentication
6. GSSAPI authentication

### Default Configuration
The [sshd_config](https://www.ssh.com/academy/ssh/sshd_config) file, responsible for the OpenSSH server, has only a few of the settings configured by default
```bash
adot8@htb[/htb]~ cat /etc/ssh/sshd_config  | grep -v "#" | sed -r '/^\s*$/d'

Include /etc/ssh/sshd_config.d/*.conf
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding yes
PrintMotd no
AcceptEnv LANG LC_*
Subsystem       sftp    /usr/lib/openssh/sftp-server
```
### Dangerous Settings
|**Setting**|**Description**|
|---|---|
|`PasswordAuthentication yes`|Allows password-based authentication.|
|`PermitEmptyPasswords yes`|Allows the use of empty passwords.|
|`PermitRootLogin yes`|Allows to log in as the root user.|
|`Protocol 1`|Uses an outdated version of encryption.|
|`X11Forwarding yes`|Allows X11 forwarding for GUI applications.|
|`AllowTcpForwarding yes`|Allows forwarding of TCP ports.|
|`PermitTunnel`|Allows tunneling.|
|`DebianBanner yes`|Displays a specific banner when logging in.|
Allowing password authentication allows us to brute-force a known username for possible passwords. Many different methods can be used to guess the passwords of users. However, some instructions and [hardening guides](https://www.ssh-audit.com/hardening_guides.html) can be used to harden our SSH servers.