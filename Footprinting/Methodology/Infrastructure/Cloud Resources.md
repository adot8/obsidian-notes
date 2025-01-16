1. Google Dork for cloud resources
	-  `intext:RBC Bank inurl:amazonaws.com`
	-  `intext:RBC Bank inurl:blob.core.windows.net`
2.  Read company website source code for assets/clues to cloud resources
3. Use [GrayHatWarfare](https://buckets.grayhatwarfare.com) to passively discover files within cloud storage
4. Use [domain.glass](https://domain.glass)  for additional infrustructure information

Even though cloud providers secure their infrastructure centrally, this does not mean that companies are free from vulnerabilities. The configurations made by the administrators may nevertheless make the company's cloud resources vulnerable. This often starts with the `S3 buckets` (AWS), `blobs` (Azure), `cloud storage` (GCP), which can be accessed without authentication if configured incorrectly.

We can use the Google Dorks `intext:` and `inurl:` to search for the company name in text and `amazonaws.com`, `blob.core.windows.net` in the URL to find company files/documents
![[Pasted image 20241231151026.png]]
```google
intext:RBC Bank inurl:amazonaws.com

intext:RBC Bank inurl:blob.core.windows.net
```

### Target Website Source
When we search for a company that we may already know or want to know, we will also come across other files such as text documents, presentations, codes, and many others.

Such content is also often included in the source code of the web pages, from where the images, JavaScript codes, or CSS are loaded. This procedure often relieves the web server and does not store unnecessary content.
![[Pasted image 20241231152215.png]]

[domain.glass](https://domain.glass) can also tell us a lot about the company's infrastructure. As a positive side effect, we can also see that Cloudflare's security assessment status has been classified as "Safe". This means we have already found a security measure that can be noted for the second layer (gateway)

![[Pasted image 20241231151903.png]]

[GrayHatWarfare](https://buckets.grayhatwarfare.com) can be used to passively discover what files are stored on a cloud storage. Many companies also use abbreviations of the company name, which are then used accordingly within the IT infrastructure. Such terms are also part of an excellent approach to discovering new cloud storage from the company. We can also search for files simultaneously to see the files that can be accessed at the same time.

> [!NOTE] Note
> Try to search for SSH keys and or passwords in these files

![[Pasted image 20241231151952.png]]