## CL.TE vulnerabilities

- Here, the **front-end server uses the** `Content-Length` header and **the back-end server uses the** `Transfer-Encoding` header.
- We can perform a simple HTTP request smuggling attack as follows:

```http
POST / HTTP/1.1  
Host: vulnerable-website.com  
Content-Length: 13  
Transfer-Encoding: chunked  
  
0  
  
SMUGGLED
```

- The front-end server processes the `Content-Length` header and determines that the request body is 13 bytes long, up to the end of `SMUGGLED`. This request is forwarded on to the back-end server.
- The back-end server processes the `Transfer-Encoding` header, and so treats the message body as using chunked encoding. It processes the first chunk, which is stated to be zero length, and so is treated as terminating the request. The following bytes, `SMUGGLED`, are left unprocessed, and the back-end server will treat these as being the start of the next request in the sequence.
- You need to include the trailing sequence `\r\n\r\n` following the final `0`.