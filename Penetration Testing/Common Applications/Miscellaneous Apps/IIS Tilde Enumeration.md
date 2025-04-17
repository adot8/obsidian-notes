### Automated
```bash
java -jar iis_shortname_scanner.jar 0 5 http://10.129.204.231/
```
If the target does not permit GET access to a file (`http://10.129.204.231/TRANSF~1.ASP,`) necessitating the brute-forcing of the remaining filename.

We can parse through big wordlists to make a smaller one that has the needed characters.
```bash
egrep -r ^transf /usr/share/wordlists/* | sed 's/^[^:]*://' > /tmp/list.txt
```

Finish her off with GoBuster
```bash
gobuster dir -u http://10.129.239.165/ -w /tmp/list.txt -x .aspx,.asp
```

### Manual
When a file or folder is created on an IIS server, Windows generates a short file name in the `8.3 format`, consisting of eight characters for the file name, a period, and three characters for the extension. Intriguingly, these short file names can grant access to their corresponding files and folders, even if they were meant to be hidden or inaccessible.

The tilde (`~`) character, followed by a sequence number, signifies a short file name in a URL. Hence, if someone determines a file or folder's short file name, they can exploit the tilde character and the short file name in the URL to access sensitive data or hidden resources.

IIS tilde directory enumeration primarily involves sending HTTP requests to the server with distinct character combinations in the URL to identify valid short file names. Once a valid short file name is detected, this information can be utilised to access the relevant resource or further enumerate the directory structure.

```http
http://example.com/~a
http://example.com/~b
http://example.com/~c
```

Assume the server contains a hidden directory named SecretDocuments. When a request is sent to `http://example.com/~s`, the server replies with a `200 OK` status code, revealing a directory with a short name beginning with "s". The enumeration process continues by appending more characters:

```http
http://example.com/~se
http://example.com/~sf
http://example.com/~sg
```

For the request `http://example.com/~se`, the server returns a `200 OK` status code, further refining the short name to "se". Further requests are sent, such as:

```http
http://example.com/~sec
http://example.com/~sed
http://example.com/~see
```

Continuing this procedure, the short name secret~1 is eventually discovered when the server returns a 200 OK status code for the request http://example.com/~secret.

For instance, if the short name `secret~1` is determined for the concealed directory SecretDocuments, files in that directory can be accessed by submitting requests such as:

```http
http://example.com/secret~1/somefile.txt
http://example.com/secret~1/anotherfile.docx

http://example.com/secret~1/somefi~1.txt
```

In 8.3 short file names, such as `somefi~1.txt`, the number "1" is a unique identifier that distinguishes files with similar names within the same directory. The numbers following the tilde (`~`) assist the file system in differentiating between files that share similarities in their names, ensuring each file has a distinct 8.3 short file name.

For example, if two files named `somefile.txt` and `somefile1.txt` exist in the same directory, their 8.3 short file names would be:

- `somefi~1.txt` for `somefile.txt`
- `somefi~2.txt` for `somefile1.txt`