
> [!Question] 
> Sometimes we can load a page in iframe, Why can we do this even there is SOP. where is the SOP protections here?

This is a very sharp observation. You are touching on one of the most common points of confusion in web security: **the difference between SOP (Same-Origin Policy) and Framing policies.**

You are 100% correct that **SOP** prevents you from _reading_ the content of the OAuth page. However, SOP does **not** prevent you from _loading_ a page in an iframe.

Here is the breakdown of why this works and where the security boundaries actually are.

### 1. SOP vs. "Framing" (Clickjacking Protection)
**The Same-Origin Policy (SOP)** says: "You can load `google.com` in an iframe on your site, but your JavaScript cannot touch the inside of that iframe."
To prevent a page from being loaded in an iframe _at all_, the server must use a specific header called **`X-Frame-Options`** or a **`Content-Security-Policy (frame-ancestors)`**.
- If the OAuth server in the lab does not send these headers, any site in the world can put that OAuth page in an iframe.
- The attacker doesn't need to "bypass" SOP to load the page; they only need to bypass it to **read** the data.
### 2. The "Handshake" that bypasses SOP
The reason this exploit works is a three-step dance where the data is "passed" from a restricted zone to an unrestricted zone:
1. **The OAuth Server (Restricted):** You iframe the login page. You cannot see the token here because of SOP.
2. **The Redirect:** After the user is "logged in" (the lab usually handles this with an active session), the OAuth server redirects the iframe to the `redirect_uri` you provided.
3. **The Proxy Page (The Leak):** The iframe is now showing `https://victim-site.com/post/comment/comment-form#token=...`.
    - Because this page is on the **Victim's** domain, it has access to its own URL (the token).
    - Because this page has that `postMessage(..., '*')` script, it **voluntarily** sends the data to you (the parent).

**The `postMessage` is the only thing bypassing SOP.** It is a "legal" bypass because the developer of the victim site explicitly wrote code to allow it.
