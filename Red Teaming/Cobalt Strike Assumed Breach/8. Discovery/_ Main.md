
Enterprise Admins
```powershell
ldapsearch "(&(objectClass=group)(sAMAccountName=Enterprise Admins))" --attributes objectSid --hostname lon-dc-1.contoso.com --dn DC=contoso,DC=com
```