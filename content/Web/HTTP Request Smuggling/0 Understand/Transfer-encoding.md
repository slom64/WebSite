The `Transfer-Encoding` header can be used to specify that the message body uses chunked encoding. This means that the message body contains one or more chunks of data. Each chunk consists of the chunk size in bytes (expressed in hexadecimal), followed by a newline, followed by the chunk contents. The message is terminated with a chunk of size zero. For example:
```http
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

b
q=smuggling
0

```
- As you can see the `\r\n` after the chunk size doesn't get counted. And after the chunk size `0` there is 2 `\r\n` one as the end of the chunck and the other as end of the request.
- If we put `empty` or `5\r\n` or `5` at the end of the request instead of `0\r\n \r\n` this will make the front/back-end timeOut. "It won't say invalid request" because it waits for `0\r\n \r\n`.
- The only way to get "`Invalid request`", is by putting data smaller or bigger the chunk size.


> [!Attention] 
> Don't calculate the `\r\n` at the end of the chunk.
