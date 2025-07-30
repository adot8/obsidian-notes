
Batch is a text-based file format used for scripting on Windows.  They contain a series of commands that are executed in order by the command line interpreter - the default on Windows is `cmd.exe`.  Batch files may have the `.bat`, `.cmd`, or `.btm` file extension.  When executed, a command prompt will open and print each command and its output (if any) as though they were being typed manually.  We typically don't want these commands being visible to the user, so a workaround is to set the `@echo` directive to `off`.  This will prevent all commands from being displayed to the user, but not their outputs.  If a command does produce output and you don't want any chance of the user seeing it, add `> nul 2>&1` to the end of the command.

A very simple BAT could look like this:

```powershell
@echo off
start payload.exe
start decoy.pdf
exit
```

A [recently discovered](https://x.com/vmray/status/1808903062926315690) trick allows a BAT file to change its behaviour depending on whether it was double-clicked or run from a command line.  This is achieved by checking the content of two shell variables:

- `%cmdcmdline%` - contains the original command line that was invoked.
    
- `%~f0` - contains the full path of the BAT file.
    

When double-clicking on a batch script, `cmdcmdline` will contain a value like `C:\Windows\system32\cmd.exe /c ""C:\Users\Daniel\Desktop\test.bat""`  and `~f0` will contain `C:\Users\Daniel\Desktop\test.bat`.  Conversely, when run from a command prompt that was opened manually, cmdcmdline will just contain `C:\Windows\System32\cmd.exe`.

The actor essentially pipes the content of cmdcmdline to see if the path of the batch file is present.  If it isn't, they assume the script was run from the command line and directs execution flow to an exit call.  Otherwise, the script will continue.  Try this example by double-clicking and running from the command line.

```powershell
@echo off
echo %cmdcmdline% | find /i "%~f0" || exit
calc
exit
```

This can defeat automated anti-virus and sandbox analysis since the script would be run programmatically, rather than interactively by a user.