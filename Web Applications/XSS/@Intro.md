| Type                                  | Description                                                                                                                                                |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Stored XSS (Persistent)               | User input is stored on the back end and then displayed when requested for (posts / comments).                                                             |
| Reflected XSS (backend processed)     | User input displays on the page after processing (browser only)                                                                                            |
| DOM based XSS (client-side processed) | Occurs when user input is directly shown in the browser and is written to an `HTML` DOM object (e.g., vulnerable username or page title).~ HTB Description |
DOM XSS - Common vulnerable Sink functions

- `document.write()`
- `DOM.innerHTML`
- `DOM.outerHTML`

The following line is an example of taking unsanitized input:
```js
document.getElementById("todo").innerHTML = "Next Task: " + decodeURIComponent(task);
```

## Manual [Payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md)​


> [!NOTE] Note
> Reading the Pages source code is very help in seeing how your input is handled. might have to add a quote here, add to the script there.Try to close the element where the input is going `'/`and get `<script></script>` in there


```js
<script>alert(window.origin)</script>
<script>prompt(window.origin)</script>
<script>print()</script>
<img src="" onerror=prompt(window.origin)>
```
```js
/index.php?PARAM=<script>prompt(window.origin)</script>
/?#task=<img src="" onerror=prompt(window.origin)>

```

View DOM based XSS in browser via CTRL+SHIFT+C

```shell
python xssstrike.py http://adot8.com/index.php?name=a
```


## Prevention

### Front-end

> [!NOTE] Note
> Input validation like the email format validation can be used
```js
function validateEmail(email) {
    const re = /^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
    return re.test($("#login input[name=email]").val());
}
```
#### Input Sanitization
We can escape special characters in input fields using the `DOMPurify` JS library
```js
<script type="text/javascript" src="dist/purify.min.js"></script>
let clean = DOMPurify.sanitize( dirty_input );
```
Never use user input directly within the following HTML tags:
- JavaScript code `<script></script>`
- CSS Style Code `<style></style>`
- Tag/Attribute Fields `<div name='INPUT'></div>`
- HTML Comments `<!-- -->`
Avoid using the following JS functions:
- `DOM.innerHTML`
- `DOM.outerHTML`
- `document.write()`
- `document.writeln()`
- `document.domain`
Avoid using the following jQuery functions as they write raw text to the HTML code:
- `html()`
- `parseHTML()`
- `add()`
- `append()`
- `prepend()`
- `after()`
- `insertAfter()`
- `before()`
- `insertBefore()`
- `replaceAll()`
- `replaceWith()`
### Back-end

> [!NOTE] Note
> Input validation like the email format validation can be used but for the back-end so it won't display the text
```php
if (filter_var($_GET['email'], FILTER_VALIDATE_EMAIL)) {
    // do task
} else {
    // reject input - do not display it
}
```
#### Input Sanitization
The `addlashes`php function can be used to add backslashes beside special characters to escape them within GET and POST requests
```php
addslashes($_GET['email'])
```
For a NodeJS back-end, we can also use the [DOMPurify](https://github.com/cure53/DOMPurify) library as we did with the front-end, as follows
```node
import DOMPurify from 'dompurify';
var clean = DOMPurify.sanitize(dirty);
```

> [!NOTE] REST
> Output HTML Encoding and using a WAF is also important

