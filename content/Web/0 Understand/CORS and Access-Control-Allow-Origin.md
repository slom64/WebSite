# CORS and the Access-Control-Allow-Origin response header

In this section we explain what the `Access-Control-Allow-Origin` header is in respect of CORS, and how it forms part of CORS implementation.

The [cross-origin resource sharing](https://portswigger.net/web-security/cors) specification provides controlled relaxation of the [[Same-origin policy (SOP)]] for HTTP requests to one website domain from another through the use of a collection of HTTP headers. Browsers permit access to responses to cross-origin requests based upon these header instructions.

---
## What is the Access-Control-Allow-Origin response header?

The `Access-Control-Allow-Origin` header is included in the response from one website to a request originating from another website, and identifies the permitted origin of the request. A web browser compares the Access-Control-Allow-Origin with the requesting website's origin and permits access to the response if they match.

---
## Implementing simple cross-origin resource sharing

The cross-origin resource sharing (CORS) specification prescribes header content exchanged between web servers and browsers that restricts origins for web resource requests outside of the origin domain. The CORS specification identifies a collection of protocol headers of which `Access-Control-Allow-Origin` is the most significant. This header is returned by a server when a website requests a cross-domain resource, with an `Origin` header added by the browser.

For example, suppose a website with origin `normal-website.com` causes the following cross-domain request:

```http
GET /data HTTP/1.1 
Host: robust-website.com
Origin : https://normal-website.com
```

The server on `robust-website.com` returns the following response:

```http
HTTP/1.1 200 OK 
... 
Access-Control-Allow-Origin: https://normal-website.com
```

The browser will allow code running on `normal-website.com` to access the response because the origins match.

The specification of `Access-Control-Allow-Origin` allows for multiple origins, or the value `null`, or the wildcard `*`. However, no browser supports multiple origins and there are restrictions on the use of the wildcard `*`.

---
## Handling cross-origin resource requests with credentials

The default behavior of cross-origin resource requests is for requests to be passed without credentials like cookies and the Authorization header. However, the cross-domain server can permit reading of the response when credentials are passed to it by setting the CORS `Access-Control-Allow-Credentials` header to true. Now if the requesting website uses JavaScript to declare that it is sending cookies with the request:

```http
GET /data HTTP/1.1
Host: robust-website.com 
...
Origin: https://normal-website.com
Cookie: JSESSIONID=<value>
```

And the response to the request is:

```http
HTTP/1.1 200 OK 
... 
Access-Control-Allow-Origin: https://normal-website.com 
Access-Control-Allow-Credentials: true
```

Then the browser will permit the requesting website to read the response, because the `Access-Control-Allow-Credentials` response header is set to `true`. Otherwise, the browser will not allow access to the response.

---
## Relaxation of CORS specifications with wildcards

The header `Access-Control-Allow-Origin` supports wildcards. For example:

```http
HTTP/1.1 200 OK 
Access-Control-Allow-Origin: *
```

> [!NOTE]
> Note that wildcards cannot be used within any other value. For example, the following header is **not** valid:
> 
> `Access-Control-Allow-Origin: https://*.normal-website.com`

Fortunately, from a security perspective, the use of the wildcard is restricted in the specification as you cannot combine the wildcard with the cross-origin transfer of credentials (authentication, cookies or client-side certificates). Consequently, a cross-domain server response of the form:

```http
HTTP/1.1 200 OK 
Access-Control-Allow-Origin: * 
Access-Control-Allow-Credentials: true
```

is not permitted as this would be dangerously insecure, exposing any authenticated content on the target site to everyone.

Given these constraints, some web servers dynamically create `Access-Control-Allow-Origin` headers based upon the client-specified origin. This is a workaround for CORS constraints that is not secure. We'll show you [how this can be exploited](https://portswigger.net/web-security/cors#server-generated-acao-header-from-client-specified-origin-header) later.

---
## Pre-flight checks

The pre-flight check was added to the CORS specification to protect legacy resources from the expanded request options allowed by CORS. Under certain circumstances, when a cross-domain request includes a non-standard HTTP method or headers, the cross-origin request is preceded by a request using the `OPTIONS` method, and the CORS protocol necessitates an initial check on what methods and headers are permitted prior to allowing the cross-origin request. This is called the pre-flight check. The server returns a list of allowed methods in addition to the trusted origin and the browser checks to see if the requesting website's method is allowed.

For example, this is a pre-flight request that is seeking to use the `PUT` method together with a custom request header called `Special-Request-Header`:

```http
OPTIONS /data HTTP/1.1
Host: <some website>
...
Origin: https://normal-website.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Special-Request-Header
```

The server might return a response like the following:

```http
HTTP/1.1 204 No Content
...
Access-Control-Allow-Origin: https://normal-website.com
Access-Control-Allow-Methods: PUT, POST, OPTIONS
Access-Control-Allow-Headers: Special-Request-Header
Access-Control-Allow-Credentials: true 
Access-Control-Max-Age: 240
```

This response sets out the allowed methods (`PUT`, `POST` and `OPTIONS`) and permitted request headers (`Special-Request-Header`). In this particular case the cross-domain server also allows the sending of credentials, and the `Access-Control-Max-Age` header defines a maximum timeframe for caching the pre-flight response for reuse. If the request methods and headers are permitted (as they are in this example) then the browser processes the cross-origin request in the usual way. Pre-flight checks add an extra HTTP request round-trip to the cross-domain request, so they increase the browsing overhead.

---
## Does CORS protect against CSRF?
- CORS does not provide protection against cross-site request forgery (CSRF) attacks, this is a common misconception.
- CORS is a controlled relaxation of the same-origin policy, so poorly configured CORS may actually increase the possibility of CSRF attacks or exacerbate their impact.
- There are various ways to perform CSRF attacks without using CORS, including simple HTML forms and cross-domain resource includes.

---

> [!Question] 
> when we are tampering resource from other domain using POST request. which mechanism prevent us from this action the Pre-flight so it stop the request or the SameSite restriction which doesn't append the user cookie so the victim won't be authenticated on the page?

It is actually **both**, but they act as different "layers" of defense. Which one stops you first depends on **how** you are sending the request.

If you are using a **script** (like `fetch` or `XMLHttpRequest`), the Pre-flight (CORS) mechanism usually kills the request before it even happens. If you are using a **standard HTML form**, the Pre-flight is skipped entirely, and the SameSite restriction becomes the primary defender.

Here is the breakdown of how these two "gatekeepers" work in tandem:

---
### Layer 1: The Pre-flight (CORS)

Triggered by: fetch(), axios, XMLHttpRequest.

Goal: To protect the server from "unexpected" cross-origin requests.

When you use JavaScript to send a `POST` request with a content type like `application/json`, the browser says: _"This is a complex request. I don't know if the target server wants to receive JSON from an external site."_
1. The browser sends an **OPTIONS** request (the Pre-flight).
2. If the server doesn't respond with `Access-Control-Allow-Origin: your-domain.com`, the browser **stops right there**.
3. The actual `POST` request is **never sent**.

---
### Layer 2: SameSite Restriction

Triggered by: Every request (Forms, fetch, `<a>` tags, `<img>`).

Goal: To prevent the browser from "leaking" session cookies to cross-site requests.

If you use a **classic HTML `<form>`**, the browser considers this a "Simple Request." Historically, forms were allowed to send data cross-site before CORS existed, so the browser **does not** send a Pre-flight for a form submission.
1. The browser prepares the `POST` request.    
2. It checks the `SameSite` attribute of the session cookie.
3. If the cookie is `Lax` (the modern default) or `Strict`, the browser **strips the cookie** from the request.1
4. The `POST` request **is sent**, but it arrives at the server without the victim's session cookie.
5. The server sees an unauthenticated request and rejects it.

---
### Comparison: Which one stops the attack?

|**Attack Method**|**Pre-flight (CORS)**|**SameSite**|**Outcome**|
|---|---|---|---|
|**`fetch()` (JSON)**|**Stops the request**|Strips cookies|Attack never reaches the server.|
|**HTML `<form>`**|_Does not apply_|**Strips cookies**|Request arrives, but unauthenticated.|
|**`fetch()` (Simple*)**|_Does not apply_|**Strips cookies**|Request arrives, but unauthenticated.|

A "Simple" fetch uses `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain` and no custom headers.

---

### Why the "2-Minute Window" is a SameSite Bypass

When you solve PortSwigger labs using the 120-second "Lax + POST" trick, you are explicitly bypassing **Layer 2**. Because you are using a `<form>`, you don't have to worry about **Layer 1** (Pre-flight).

The 2-minute window tells the browser: _"I know this cookie is Lax, but since it's brand new, I'll pretend it's SameSite=None just this once."_ This allows the cookie to be appended to your malicious form submission.
