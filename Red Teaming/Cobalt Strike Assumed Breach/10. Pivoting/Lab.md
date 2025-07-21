- On the Attacker Desktop, run Terminal as a local admin.
    
-  Add the following static DNS entry:
    
    TerminalTypeCopy
    
    `Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value '10.10.120.20 lon-db-1'`
    
-  Open Cobalt Strike and connect to the team server.
    
-  Use Beacon to start a SOCKS 5 proxy on port 1080.
    
    1.`socks 1080 socks5`