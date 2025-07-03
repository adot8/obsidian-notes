This is a similar technique to HTML smuggling but leverages the SVG format instead.  SVG, or Scalable Vector Graphics, is an XML-based vector image format for defining two-dimensional graphics and animations.  A very simple SVG can be defined like so:

```html
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
   <circle cx="50" cy="50" r="40" stroke="black" stroke-width="4" fill="none" />
   Sorry, your browser does not support inline SVG.
</svg> 
```

Amazingly, SVG allows embedded JavaScript.

```html
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
<circle cx="50" cy="50" r="40" stroke="black" stroke-width="4" fill="none" />
<script>
    alert('Hello World');
</script>
Sorry, your browser does not support inline SVG.
</svg> 
```

SVGs can be embedded into HTML or saved as standalone files.  This makes it possible to use the `.svg` file format as a payload container, as the JavaScript will trigger if a browser, such as Edge, is the default file handler for this extension.