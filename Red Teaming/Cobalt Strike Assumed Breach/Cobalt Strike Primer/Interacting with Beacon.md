When a new Beacon session first checks into the team server, a record will appear in the Event Log (**View > Event Log**) and a new session will appear in the table or graph view, depending on which one is selected.

![[Pasted image 20250630095240.png]]

The columns in the table view are fairly self-explanatory, but here's a summary:

- **external** is the external IP address of the target.  This is resolved by the Cobalt Strike's web server, not sent by Beacon itself.
- **internal** is the internal IP address of the computer.
- **listener** is the name of the egress listener the Beacon is communicating through.
- **user** is the username the Beacon process is running as.
- **computer** is the hostname of the computer.
- **note** is where you can add your own notes.
- **process** is the name of the process the Beacon is running in.
- **pid** is the process ID the Beacon is running in.
- **arch** is the Beacon architecture, either x64 or x86.
- **last** is the last time, in ms, s, m, h, or d, the Beacon communicated with the team server.
- **sleep** is the sleep time of the Beacon.

The blue monitor icon denotes that the Beacon is running in medium-integrity (i.e. standard user privileges).  When running in high-integrity (i.e. local administrator or SYSTEM privileges), a red monitor icon is shown and the username as appended with an asterisk.

![[Pasted image 20250630095453.png]]
Beacon identifies itself to the team server by sending its metadata within the C2 protocol.  For HTTP/S, this is somewhere within the GET or POST request such as the URL, a header, or cookie.  The metadata itself is encrypted using the public key of the team server from which the payload was generated.  This prevents a Beacon from communicating with, or being tasked by, a different team server.

To 'interact' with a Beacon, double-click on its row or right-click and select **Interact**.  This will open a new tab.

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/1e3c613134964b30f6f4d4d48ceb6c9d.png)

For a list of commands, type `help` in the text box at the bottom and press enter.  This will display a short summary of each command.

```powershell
beacon> help

beacon> help getuid
Use: getuid

Prints the User ID associated with the current token
```

To task the Beacon, simply type a valid command and press enter again.  This will place a job in the queue.

> Multiple commands can be queued whilst you wait for the Beacon to check-in.  The queue can be cleared with the `clear` command.  If jobs are available when Beacon checks in, the team server will provide them in the traffic response.  The client also prints a small log to indicate that jobs were sent.

Beacon will process each job in the order they were queued and send the results back.  The client then displays the output to the operator.
```powershell
[02/26 15:52:55] beacon> getuid
[02/26 15:52:55] [*] Tasked beacon to get userid
[02/26 15:52:59] [+] host called home, sent: 8 bytes
[02/26 15:52:59] [*] You are DESKTOP-FGSTPS7\Attacker
```

### Session Graph View

The image below shows one egress and two P2P Beacons, but it's not immediately obvious which Beacon is connected to which.  Are both P2P Beacons connected directly to the HTTP Beacon, or are they arranged in a different way?

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/45b35d6d367a44ce70954272c87b5f3a.png)

This is where the graph view comes into play.  Although it shows less information than the table view, the relationship between each Beacon comes obvious.

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/4cb215a4d8a67be404045fd775ea6fde.png)

The firewall icon and dashed line represent egress Beacons that are talking directly out to the team server.  The solid lines represent P2P connections.  The colours are also representative of the protocol in use. They are:

- Dashed green for HTTP/S.
    
- Solid yellow for SMB.
    
- Dashed yellow for DNS.
    
- Solid green for TCP.
    

Right-click somewhere on the black background to bring up a menu containing alternate layout options.  The default layout seen here is **Tree > Left**.  You may also right-click on a Beacon to bring up the same interaction menu as you see in the table view.