We may be able to escalate privileges on well-patched and well-configured systems if users are permitted to install software or vulnerable third-party applications/services are used throughout the organization. It is common to encounter a multitude of different applications and services on Windows workstations during our assessments. Let's look at an instance of a vulnerable service that we could come across in a real-world environment. Some services/applications may allow us to escalate to SYSTEM. In contrast, others could cause a denial-of-service condition or allow access to sensitive data such as configuration files containing passwords.

#### Exploitation
Find installed programs
```powershell
wmic product get name
```

`Druva inSync` application stands out. A quick Google search shows that version `6.6.3` is vulnerable to a command injection attack via an exposed RPC service.

We can use [this](https://www.exploit-db.com/exploits/49211) exploit PoC to escalate our privileges

Enumerate local ports
```powershell
netstat -ano | findstr 6064
get-process -Id 3324
```

Enumerate running services
```powershell
get-service | ? {$_.DisplayName -like 'Druva*'}
```