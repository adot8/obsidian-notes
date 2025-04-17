
> [!NOTE] Note
> Take note of the Banner displayed of the FTP version. Exploits??

```bash
ftp> ls -R
ftp> debug
ftp> trace


wget -m --no-passive ftp://anonymous:anonymous@10.10.10.184/

ftp-ssl -z secure -z verify=0 -p $ip
ftp-ssl -z secure -z verify=0 $ip -z cipher="$(openssl ciphers -tls1)" -p $ip

openssl s_client -connect 10.129.14.136:21 -starttls ftp

hydra -l admin -P ~/opt/wordlists/2023-200_most_used_passwords.txt ftp://192.168.1.100
```


Client and server establish a control channel through `TCP Port 21`. A data channel can then be established over `TCP Port 20`.

Two modes `Active` and `Passive` 

n the active variant, the client establishes the connection as described via TCP port 21 and thus informs the server via which client-side port the server can transmit its responses. However, if a firewall protects the client, the server cannot reply because all external connections are blocked

For this purpose, the `passive mode` has been developed. Here, the server announces a port through which the client can establish the data channel. Since the client initiates the connection in this method, the firewall does not block the transfer.

### TFTP 
`Trivial File Transfer Protocol` functions over UDP, doesn't support authentication and solely operations based on read write permissions.

| **Commands** | **Description**                                                                                                                        |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `connect`    | Sets the remote host, and optionally the port, for file transfers.                                                                     |
| `get`        | Transfers a file or set of files from the remote host to the local host.                                                               |
| `put`        | Transfers a file or set of files from the local host onto the remote host.                                                             |
| `quit`       | Exits tftp.                                                                                                                            |
| `status`     | Shows the current status of tftp, including the current transfer mode (ascii or binary), connection status, time-out value, and so on. |
| `verbose`    | Turns verbose mode, which displays additional information during file transfer, on or off.                                             |
### vsFTPd
vsFTPd is the most used FTP server on Linux.

The configuration file can be found at `/etc/vsftpd.conf` along with the `/etc/ftpusers` that we also need to pay attention to, as this file is used to deny certain users access to the FTP service


| **Setting**                                                   | **Description**                                                                                          |
| ------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `listen=NO`                                                   | Run from inetd or as a standalone daemon?                                                                |
| `listen_ipv6=YES`                                             | Listen on IPv6 ?                                                                                         |
| `anonymous_enable=NO`                                         | Enable Anonymous access?                                                                                 |
| `local_enable=YES`                                            | Allow local users to login?                                                                              |
| `dirmessage_enable=YES`                                       | Display active directory messages when users go into certain directories?                                |
| `use_localtime=YES`                                           | Use local time?                                                                                          |
| `xferlog_enable=YES`                                          | Activate logging of uploads/downloads?                                                                   |
| `connect_from_port_20=YES`                                    | Connect from port 20?                                                                                    |
| `secure_chroot_dir=/var/run/vsftpd/empty`                     | Name of an empty directory                                                                               |
| `pam_service_name=vsftpd`                                     | This string is the name of the PAM service vsftpd will use.                                              |
| `rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem`          | The last three options specify the location of the RSA certificate to use for SSL encrypted connections. |
| `rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key` |                                                                                                          |
| `ssl_enable=NO`                                               |                                                                                                          |
#### Dangerous Settings
|**Setting**|**Description**|
|---|---|
|`anonymous_enable=YES`|Allowing anonymous login?|
|`anon_upload_enable=YES`|Allowing anonymous to upload files?|
|`anon_mkdir_write_enable=YES`|Allowing anonymous to create new directories?|
|`no_anon_password=YES`|Do not ask anonymous for password?|
|`anon_root=/home/username/ftp`|Directory for anonymous.|
|`write_enable=YES`|Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE?|
#### Good Settings
|**Setting**|**Description**|
|---|---|
|`hide_ids=YES`|All user and group information in directory listings will be displayed as "ftp".|
This setting is a security feature to prevent local usernames from being revealed. With the usernames, we could attack the services like FTP and SSH and many others with a brute-force attack in theory