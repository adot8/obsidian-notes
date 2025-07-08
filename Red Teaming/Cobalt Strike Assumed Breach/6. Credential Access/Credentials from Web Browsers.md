
Whilst many web-based applications can be configured for single sign-on, and therefore authenticate users automatically, some may still require plaintext credentials.  Users commonly allow their browsers to save their credentials so they don't have to type them in every time.

Browsers will typically encrypt these credentials using the [Windows Data Protection API](https://learn.microsoft.com/en-us/windows/win32/api/dpapi/) (DPAPI) and store them in a local database.  Adversaries may be able to read and decrypt this database to obtain the original plaintext passwords [T1555.003](https://attack.mitre.org/techniques/T1555/003/).

The specific procedure may vary depending on the browser being used, because each have their own implementations.  Most Chromium-based browsers keep a SQLite database in the path `%LOCALAPPDATA%\<vendor>\<browser>\User Data\Default\Login Data`.

For example, Chrome stores its database at `%LOCALAPPDATA%\Google\Chrome\User Data\Default\Login Data`.

Tools like SharpChrome can automatically read and decrypt credentials from the database.

```powershell
beacon> execute-assembly C:\Tools\SharpDPAPI\SharpChrome\bin\Release\SharpChrome.exe logins
```

![[Pasted image 20250708092330.png]]

> This technique can be used from a medium-integrity context.