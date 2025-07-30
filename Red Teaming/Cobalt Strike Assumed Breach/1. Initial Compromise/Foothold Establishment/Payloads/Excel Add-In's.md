## Excel Add-In's

An ***.xlam** file is a macro-enabled Excel Add-In, designed to add custom functionality to Excel.  The typical use case is to bind a macro to a button on the ribbon to provide a shortcut for executing common tasks.  The main challenge with using macros for initial access is Protected View - this disables macros when opening a file that has MotW (discussed in the next lesson), was received as an email attachment, or when opened from an untrusted location.  We can strip MotW when using an appropriate container (explained later in this chapter), and we can also get around the trusted location issue.  By default, the following locations are included in the Trust Center:

- %ProgramFiles%\Microsoft Office\root\Office16\Library\
    
- %ProgramFiles%\Microsoft Office\root\Office16\STARTUP\
    
- %ProgramFiles%\Microsoft Office\root\Office16\XLSTART\
    
- %ProgramFiles%\Microsoft Office\root\Templates\
    
- %APPDATA%\Microsoft\Excel\XLSTART\
    
- %APPDATA%\Microsoft\Templates\

We cannot write into any of the `%ProgramFiles%` directories as a standard user, but we can write into `%APPDATA%`.  Furthermore, Excel will automatically load any workbooks, templates, or add-in's present in the XLSTART directory.

To create an XLAM, create a new empty workbook and open the VB editor using **Alt + F11**.  Right-click on the VBAProject and select **Insert > Module**.

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/d28a4c0f30daecf479e6454c92d71249.png)

Then add your macro code into the new module.  For example:
```vb
Private Sub Auto_Open()
   MsgBox "Hello World", vbOKOnly, "pwned"
End Sub
```

Close the editor, go to **File > Save As**, and save the workbook as an **Excel Add-In (*.xlam)** file.

>When you select the Add-In type, it will automatically switch the save location to `C:\Users\Attacker\AppData\Roaming\Microsoft\AddIns`.  I suggest you save it somewhere such as `C:\Payloads`.

To test it, copy the XLAM file into `%APPDATA%\Microsoft\Excel\XLSTART`, then close and re-open Excel.  The message box should appear straight away.

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/71a3643d5a0596ed00d5080e35cb0fef.png)
