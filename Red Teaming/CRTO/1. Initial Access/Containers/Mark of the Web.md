Mark of the Web, aka MotW, is a zone identifier used to mark files that have been downloaded from the Internet as potentially unsafe.  This can be seen on a file by looking at its properties in Explorer or using PowerShell.

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/e2924df834b666c79c4d677ad723d63f.png)

```powershell
PS C:\Users\Attacker\Downloads> Get-Content -Stream Zone.Identifier .\test.pdf
[ZoneTransfer]
ZoneId=3
ReferrerUrl=https://s28.q4cdn.com/392171258/files/doc_downloads/test.pdf
HostUrl=https://s28.q4cdn.com/392171258/files/doc_downloads/test.pdf
```

MotW is troublesome when phishing because Windows may present additional security warnings to the user when attempting to open or run files that have it.  Some files, such as Office documents, will not enable macros if MotW is present.

Containers provide a means of bundling your dependencies (trigger, payload, and decoy) into a single file.  This simplifies the process of sending multiple files to a victim and can add a level of obfuscation (e.g. if they can be password protected).  The ISO/IMG, ZIP, and WIM formats are solid choices as they're natively supported by Windows.  You could go for something like 7z, Gz, or WinRAR, but a victim may not have the required software available to interact with them.  You can package your files manually, or use a tool such as mgeeky's [PackMyPayload](https://github.com/mgeeky/PackMyPayload).  Some of these container formats support hidden files, and some do not propagate MotW.  [This repository](https://github.com/nmantani/archiver-MOTW-support-comparison) by [Nobutaka Mantani](https://x.com/nmantani) contains a comparison of MotW propagations.

For example, I can pack the aforementioned PDF executable into an ISO.

```bash
attacker@DESKTOP-FGSTPS7:/mnt/c/Users/Attacker/Downloads$ /mnt/c/Tools/PackMyPayload/PackMyPayload.py test.pdf test.iso

[.] Packaging input file to output .iso (iso)...
Burning file onto ISO:
    Adding file: /test.pdf
[+] File packed into ISO.

[+] Generated file written to (size: 65536): test.iso
```

We can then test it by hosting it on a Python web server, then pulling it down via PowerShell.

Then when it's mounted, we can see that MotW is no longer applied to the PDF.

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/a4bf46c21cc2ea1da5dbd6752b7e0bf9.png)

PackMyPayload also has the option to set the hidden attribute on files as they're packaged into a container.  This is useful for scenarios where you want to hide files such as the decoy and payload, so that the user only sees the trigger.

For example, if we had three files, a trigger, payload, and decoy:

```bash
$ ls -l /mnt/c/Payloads/xlam

-rwxrwxrwx 1 rasta rasta 11607 Jun 27 13:55 decoy.xlsx
-rwxrwxrwx 1 rasta rasta 12906 Jun 27 13:55 payload.xlam
-rwxrwxrwx 1 rasta rasta  2094 Jun 28 13:55 trigger.xls.lnk
```

We can pack them into an IMG file whilst hiding _decoy.xlsx_ and _payload.xlam_ using the `-H` parameter.

```bash
$ python3 PackMyPayload.py -H decoy.xlsx,payload.xlam /mnt/c/Payloads/xlam /mnt/c/Payloads/xlam/package.img
```