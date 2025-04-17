### Powershell
```powershell
gci \\192.168.220.129\Finance\
```

Add new drive
```powershell
New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem"
```

Add drive with credentials
```powershell
PS C:\htb> $username = 'plaintext'
PS C:\htb> $password = 'Password123'
PS C:\htb> $secpassword = ConvertTo-SecureString $password -AsPlainText -Force
PS C:\htb> $cred = New-Object System.Management.Automation.PSCredential $username, $secpassword
PS C:\htb> New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem" -Credential $cred
```

List amount of files
```powershell
PS C:\htb> N:
PS N:\> (Get-ChildItem -File -Recurse | Measure-Object).Count
```

Search for file name
```powershell
Get-ChildItem -Recurse -Path N:\ -Include *cred* -File
```

Search for words within file
```powershell
Get-ChildItem -Recurse -Path N:\ | Select-String "cred" -List
```

### CMD

List amount of files on a share
```powershell
net use n: \\192.168.220.129\Finance /user:plaintext Password123
dir n: /a-d /s /b | find /c ":\"
```

| `/a-d` | `/a` is the attribute and `-d` means not directories           |
| ------ | -------------------------------------------------------------- |
| `/s`   | Displays files in a specified directory and all subdirectories |
| `/b`   | Uses bare format (no heading information or summary)           |
Search for file names in file share
```powershell
dir n:\*cred* /s /b
dir n:\*secret* /s /b
```

Search for credentials
```powershell
findstr /s /i cred n:\*.*
```

### Linux
Connecting to share
```bash
sudo mkdir /mnt/Finance

sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance
```

Alternatively - use a credential file
```bash
username=plaintext
password=Password123
domain=.
```

```bash
mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile
```

Find file names
```bash
find /mnt/Finance/ -name *cred*
```