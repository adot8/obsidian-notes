
All of the [lateral movement](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=674b793699b3a08430017d88) techniques that we covered rely on Windows protocols that are designed to facilitate remote administration.  The primary reason for this is to blend lateral movement activity into legitimate administrative activity, which makes it harder to detect.  These protocols rely on different Windows services, which also means they require different service tickets to access.

All of the examples in this chapter demonstrated obtaining a CIFS service ticket which grants access to the SMB service and list the C$ drive of the target.  This is generally okay as a means of demonstrating an 'attack' worked but has limited utility.  The aim of this page is simply to provide some guidance on what service tickets are required to access services that are useful for lateral movement or data access.

|   |   |   |
|---|---|---|
|**Name**|**Description**|**Ticket(s)**|
|SMB|Access the remote filesystem.  View, list, upload, & delete files.|CIFS|
|PsExec|Run a binary via the Service Control Manager.|CIFS|
|WinRM|Windows Remote Management.|HTTP|
|WMI|Execute applications on the remote target, e.g. process call create.|RPCSS  <br>HOST  <br>RestrictedKrbHost|
|RDP|Remote Desktop Protocol.|TERMSRV  <br>HOST|
|MSSQL|MS SQL Databases.|MSSQLSvc|
