An SSP (Security Support Provider) is a DLL that allows for authentication in applications via **NTLM, Kerberos, Wdigest and CredSSP**

Mimikatz provides a custom SSP - `mimilib.dll` This SSP logs local logons,
service account and machine account passwords in clear text on the
target server.