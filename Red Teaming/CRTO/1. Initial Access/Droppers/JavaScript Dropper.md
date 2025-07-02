
This lesson will demonstrate how to create a JavaScript dropper out of a .NET assembly, using [GadgetToJScript](https://github.com/med0x2e/GadgetToJScript).  Start off by creating a new .NET Framework Class Library project in Visual Studio.

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/dbd94b0976ecdf2e5bcd96a2a37ae697.png)

> Make sure you pick the .NET Framework version, rather than .NET Core.

The project will be created with an empty `Class1.cs` file.

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
  
namespace MyDropper
{
    public class Class1
    {
    }
}
```

First, rename `Class.cs` to `Dropper.cs` (not required, but kinda nice).

Then, right-click on the project in the Solution Explorer and go to **Add > Existing Item**.  Select a payload file to embed in the dropper, e.g. `http_x64.exe`, then in its properties, change the **Build Action** to **Embedded Resource**.

