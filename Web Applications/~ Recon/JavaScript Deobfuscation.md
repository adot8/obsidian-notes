https://jsconsole.com/
## Obfuscation
1. https://www.toptal.com/developers/javascript-minifier
2. https://beautifytools.com/javascript-obfuscator.php
3. https://obfuscator.io/
4. https://jsfuck.com/
5. https://utf-8.jp/public/jjencode.html
6. https://utf-8.jp/public/aaencode.html
## Deobfuscation

> [!NOTE] Note
> View `script.js` or `.js` files and see what theyre about 

1. Cleanup / Beautify
	1. https://beautifier.io/
	2. https://prettier.io/playground/
2. UnPack
	1. https://matthewfl.com/unPacker.html
3. Reverse Engineer and interpret
## Decoding
Hex
```shell
echo https://www.hackthebox.eu/ | xxd -p
echo 68747470733a2f2f7777772e6861636b746865626f782e65752f0a | xxd -p -r
```
Rot13
```shell
echo https://www.hackthebox.eu/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'
echo uggcf://jjj.unpxgurobk.rh/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```
https://www.boxentriq.com/code-breaking/cipher-identifier