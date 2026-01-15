## TE.CL vulnerabilities

- the front-end server uses the `Transfer-Encoding` header and the back-end server uses the `Content-Length` header. We can perform a simple HTTP request smuggling attack as follows:
```
POST / HTTP/1.1  
Host: vulnerable-website.com  
Content-Length: 3  
Transfer-Encoding: chunked  
  
8  
SMUGGLED  
0
```

- The front-end server processes the `Transfer-Encoding` header, and so treats the message body as using chunked encoding. It processes the first chunk, which is stated to be 8 bytes long, up to the start of the line following `SMUGGLED`. It processes the second chunk, which is stated to be zero length, and so is treated as terminating the request. This request is forwarded on to the back-end server.
- The back-end server processes the `Content-Length` header and determines that the request body is 3 bytes long, up to the start of the line following `8`. The following bytes, starting with `SMUGGLED`, are left unprocessed, and the back-end server will treat these as being the start of the next request in the sequence.
- You need to include the trailing sequence `\r\n\r\n` following the final `0`.

- in TE.CL you append the next request by your own but it will be executed by requesting new request, or you can smuggle other users requests.

![[Z Assets/Images/Pasted image 20260112173758.jpeg]]

- don't sum the `\r\n` before the 0 in the smuggled request, it used for the end of chunk. if you sum it, this will error in front-end server.
- the `content-length` in the smuggled request is too big, so the rest of the next request will be treated as body of this request.