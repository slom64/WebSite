The `Content-Length` header is straightforward: it specifies the length of the message body in bytes. For example:
```http
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 13

q=smuggling

```
- As you can see, The `\r\n` between the headers and Post body doesn't get counted 
- The `\r\n` at the end of the body get counted.