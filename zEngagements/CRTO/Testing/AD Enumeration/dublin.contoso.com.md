
#### Workstation Admins

```powershell
\\dublin.contoso.com\SysVol\dublin.contoso.com\Policies\{E9591535-E913-4430-8384-91DA959E2194}
```

```powershell
DUB-WKSTN-1$
DUB-WKSTN-2$
```

```powershell
[Unicode]
Unicode=yes
[Version]
signature="$CHICAGO$"
Revision=1
[Group Membership]
*S-1-5-21-2958544638-1589230383-838459903-3103__Memberof = *S-1-5-32-544
*S-1-5-21-2958544638-1589230383-838459903-3103__Members =

```

![[Pasted image 20250731092558.png]]

```powershell
tdobson
etate
```

---

### Constrained Delegation

```
DUB-WEB-1$
S-1-5-21-2958544638-1589230383-838459903-2601
```

---

### Web Admins

```powershell
\\dublin.contoso.com\SysVol\dublin.contoso.com\Policies\{BC922AEC-8191-4B0D-8592-8C483703D7FD}
```

```xml
[Unicode]
Unicode=yes
[Version]
signature="$CHICAGO$"
Revision=1
[Group Membership]
*S-1-5-21-2958544638-1589230383-838459903-1605__Memberof = *S-1-5-32-544
*S-1-5-21-2958544638-1589230383-838459903-1605__Members =
```

```powershell
Web Admins
S-1-5-21-2958544638-1589230383-838459903-1605
```

---

### AppLocker on all WS

---

### Domain Admins

Most active
```powershell
nwallace
S-1-5-21-2958544638-1589230383-838459903-4102
```