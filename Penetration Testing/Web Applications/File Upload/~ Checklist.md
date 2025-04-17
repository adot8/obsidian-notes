1. Bypass client side verification and upload malicious doc
2. Attempt to upload SVG for XXE and read source code to understand naming scheme and locate upload directory
3. Fuzz for double-extensions and special character bypasses
4. Upload image and use magic bytes to bypass security measures
5. If no XXE fuzz for upload directory