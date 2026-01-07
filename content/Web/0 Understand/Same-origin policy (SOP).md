## What is the same-origin policy?

The same-origin policy is a web browser security mechanism that aims to prevent websites from attacking each other.

The same-origin policy restricts scripts on one origin from accessing data from another origin. An origin consists of a URI scheme, domain and port number. For example, consider the following URL:

```
http://normal-website.com/example/example.html
```

This uses the scheme `http`, the domain `normal-website.com`, and the port number `80`. The following table shows how the same-origin policy will be applied if content at the above URL tries to access other origins:

| URL accessed                              | Access permitted?                  |
| ----------------------------------------- | ---------------------------------- |
| `http://normal-website.com/example/`      | Yes: same scheme, domain, and port |
| `http://normal-website.com/example2/`     | Yes: same scheme, domain, and port |
| `https://normal-website.com/example/`     | No: different scheme and port      |
| `http://en.normal-website.com/example/`   | No: different domain               |
| `http://www.normal-website.com/example/`  | No: different domain               |
| `http://normal-website.com:8080/example/` | No: different port*                |

*Internet Explorer will allow this access because IE does not take account of the port number when applying the same-origin policy.

## Why is the same-origin policy necessary?

When a browser sends an HTTP request from one origin to another, any cookies, including authentication session cookies, relevant to the other domain are also sent as part of the request. This means that the response will be generated within the user's session, and include any relevant data that is specific to the user. Without the same-origin policy, if you visited a malicious website, it would be able to read your emails from GMail, private messages from Facebook, etc.

## How is the same-origin policy implemented?

The same-origin policy generally controls the access that JavaScript code has to content that is loaded cross-domain. Cross-origin loading of page resources is generally permitted. For example, the SOP allows embedding of images via the `<img>` tag, media via the `<video>` tag, and JavaScript via the `<script>` tag. However, while these external resources can be loaded by the page, any JavaScript on the page won't be able to read the contents of these resources.

There are various exceptions to the same-origin policy:

- Some objects are writable but not readable cross-domain, such as the `location` object or the `location.href` property from iframes or new windows.
- Some objects are readable but not writable cross-domain, such as the `length` property of the `window` object (which stores the number of frames being used on the page) and the `closed` property.
- The `replace` function can generally be called cross-domain on the `location` object.
- You can call certain functions cross-domain. For example, you can call the functions `close`, `blur` and `focus` on a new window. The `postMessage` function can also be called on iframes and new windows in order to send messages from one domain to another.

Due to legacy requirements, the same-origin policy is more relaxed when dealing with cookies, so they are often accessible from all subdomains of a site even though each subdomain is technically a different origin. You can partially mitigate this risk using the `HttpOnly` cookie flag.

It's possible to relax same-origin policy using `document.domain`. This special property allows you to relax SOP for a specific domain, but only if it's part of your FQDN (fully qualified domain name). For example, you might have a domain `marketing.example.com` and you would like to read the contents of that domain on `example.com`. To do so, both domains need to set `document.domain` to `example.com`. Then SOP will allow access between the two domains despite their different origins. In the past it was possible to set `document.domain` to a TLD such as `com`, which allowed access between any domains on the same TLD, but now modern browsers prevent this.


---
## Before SameSite

> [!Question] 
> if SameSite restriction was not made yet. then we can bypass the SOP by using redirections like window.location = "https://asdf.com" but we send malicious parameters. which will include the use cookie

You have perfectly connected the dots. If we rewind to the time before **SameSite** became the default, your logic is exactly how the web was exploited for years.

However, there is a tiny but critical distinction in your phrasing: you aren't exactly "bypassing" SOP; you are **sidestepping** it.

### The "Sidestep" Strategy

When you use `window.location = "https://vulnerable.com/change-email?email=hacker@evil.com"`, you are performing a **Top-Level Navigation**.

- **SOP's Perspective:** SOP doesn't care about this because your malicious script is essentially "suiciding." Once the window navigates to the new URL, your script is gone. You aren't trying to _read_ the response anymore; you've just forced the browser to _visit_ a new page.
- **The Cookie's Perspective:** Because it’s a top-level navigation to the target domain, the browser says, "The user is now going to vulnerable.com, so I must include their cookies."

---

### Why this is the "Ultimate" CSRF (The GET Vulnerability)

If a website allows sensitive actions (like changing an email or a password) via a `GET` request, it is incredibly easy to exploit.

The "Unnoticeable" Way (No Redirect Needed):

Before SameSite, you didn't even need to redirect the user's whole window. You could just do this:

```html
<img src="https://vulnerable.com/change-email?email=hacker@evil.com" style="display:none;">
```

The browser would try to "load the image," send the request to the server with the victim's cookies, the email would change, and the user would never even know a request was made. **SOP cannot stop an `<img>` or `<script>` tag from making a cross-origin request.**

---

### The Modern Reality: Lax + GET

This is why **SameSite=Lax** was invented. It specifically protects against the scenario you described:
1. **Lax allows GET:** If you click a link or use `window.location`, the cookies **are** sent (to ensure you stay logged in when clicking a link from Google to your Bank).
2. **Lax blocks Sub-resources:** If you try to use the `<img>` tag trick mentioned above, Chrome will **refuse** to send the session cookie because an `<img>` tag is not a "Top-Level" navigation.
3. **Lax blocks POST:** If you try to submit a hidden form, the cookies are stripped.

---

You are 100% correct: **SOP was never meant to stop CSRF.** SOP stops you from _stealing_ data (Reading). CSRF is about _acting_ as the user (Writing). Until `SameSite` came along, the web was essentially wide open to "Write" attacks as long as the attacker didn't care about reading the response.


---
## CORS vs SameSite

> [!question] 
> so if we are in allowed origin, and we are running javascript code that fetch resources from other origin, that mean the browser will put the cookie of the victim even if the cookie SameSite strict

Actually, no—and this is a very common point of confusion. If a cookie is set to **`SameSite=Strict`**, the browser will **never** include that cookie in a request initiated by a script (like `fetch` or `XMLHttpRequest`) if the script is running on a different domain, **even if the server's CORS policy "allows" that origin.**

Here is the breakdown of why that happens and how the layers of defense work together.

### 1. The Conflict: CORS vs. SameSite
It helps to think of CORS and SameSite as two different security guards at two different gates:
- **Gate 1: SameSite (The Cookie Guard):** This guard lives in the browser. It looks at the **URL in the address bar** vs. the **URL being requested**. If they don't match exactly and the cookie is `Strict`, the guard stops the cookie from leaving the browser.
- **Gate 2: CORS (The Origin Guard):** This guard lives on the target server. It looks at the `Origin` header of the incoming request. If the origin is "allowed," it tells the browser, "You can let the script read my response."
### 2. Why "Strict" blocks the request
If you are on `attacker.com` and you run:
JavaScript
```
fetch('https://bank.com/api/user', { credentials: 'include' });
```
1. The browser sees the request is going to `bank.com`.
2. The browser looks for `bank.com` cookies.
3. It finds a cookie marked **`SameSite=Strict`**.
4. The browser says: _"The current site is attacker.com, but the cookie is Strict for bank.com. I will NOT attach this cookie."_
5. The request arrives at `bank.com` **without any session cookie**.
6. The bank responds with `401 Unauthorized`.

**It doesn't matter if the Bank's CORS policy says `Access-Control-Allow-Origin: https://attacker.com`**. CORS only gives the script permission to **read** the response; it does not have the power to "force" a `Strict` cookie to be sent cross-site.

---

### 3. Comparison of Scenarios

|**Cookie Attribute**|**fetch() from attacker.com**|**Top-level link from attacker.com**|
|---|---|---|
|**`SameSite=None`**|Cookie is sent (if `credentials: include`)|Cookie is sent|
|**`SameSite=Lax`**|**Cookie is NOT sent**|Cookie is sent|
|**`SameSite=Strict`**|**Cookie is NOT sent**|**Cookie is NOT sent**|

> **Note:** With `Strict`, even if you click a link from your email (Gmail) to your Bank, you will be logged out when the page loads because it originated from a different site. You have to refresh or re-navigate to get the session back.

---

### 4. When DOES the cookie get sent?

The **only** way a `Strict` cookie gets sent is if the "Site for Cookies" (the domain in the address bar) matches the target.

This is why **Cross-Site WebSocket Hijacking** usually only works if the cookies are `SameSite=None` or sometimes `Lax`, but almost never if they are `Strict`.

### Summary

If a cookie is **`SameSite=Strict`**, you cannot perform a CSRF attack using `fetch`, even if the target site has a misconfigured (wide-open) CORS policy. The browser's cookie protection triggers _before_ the server's CORS policy is even evaluated.

---
## Bypass SameSite + CORS

> [!question] 
> so even if we are in allowed domain in SOP, we need to bypass the SameSite restrictions by doing get request from top level function, or doing POST request targeting cookies that have been set in less than 2 minutes 

Exactly. You have synthesized the two main "cracks" in modern browser defenses perfectly.

To be technically precise: **CORS (Allowed Origins)** determines if a script can **read** the data, but **SameSite** determines if the browser will **send** the credentials (cookies) in the first place.

Here is the final breakdown of those two bypasses you mentioned:

### 1. The "Lax + POST" (The 2-Minute Window)

This is a specific "grace period" implemented in Chromium-based browsers (Chrome, Edge, Brave).
- **The Rule:** If a cookie does not have a `SameSite` attribute defined, the browser defaults to `Lax`.
- **The Exception:** For the first **120 seconds** after a cookie is set, the browser will treat it as `SameSite=None` for top-level `POST` requests.
- **The Attack:** You force a cookie refresh (e.g., via a social login or a popup) and then immediately trigger a `<form>.submit()`.

### 2. The Top-Level GET (The Navigation Bypass)

`SameSite=Lax` (the modern default) allows cookies to be sent on **GET** requests, but only if they are **Top-Level Navigations**.
- **Allowed:** `window.location = "..."` or `<a href="...">`.
- **Blocked:** `fetch("...")`, `<img>`, or `<iframe>`.
- **The Attack:** If a sensitive action (like `change-email`) can be triggered via a `GET` parameter, you simply redirect the victim's entire tab to that URL.    

---

### The Only Way to Bypass `SameSite=Strict`

If a developer sets a cookie to `Strict`, **neither** of the above tricks will work. The browser will not send a `Strict` cookie if the request originates from a different domain, period.

To bypass `Strict`, you usually need a **secondary vulnerability**:
1. **On-Site XSS:** If you find an XSS vulnerability on the _same domain_ (e.g., `vulnerable.com`), your script is now running on a "Same-Site" context. The browser will happily include `Strict` cookies.
2. **Subdomain Takeover:** If you control `sub.vulnerable.com`, you are often considered "Same-Site" (depending on the Public Suffix List), allowing you to bypass `Strict` restrictions.    

---

### Summary Table for your CTF Notes

|**Technique**|**Works on None?**|**Works on Lax?**|**Works on Strict?**|
|---|---|---|---|
|**Classic `<form>` CSRF**|Yes|No|No|
|**2-Minute Window POST**|Yes|**Yes** (if fresh)|No|
|**Top-Level GET Redirect**|Yes|**Yes**|No|
|**On-Site XSS**|Yes|Yes|**Yes**|

### Pro-Tip:

If you are stuck and see `SameSite=Strict` in the headers, look for a way to perform **"Request Tunnelling"** or find a **Client-side redirect** on the target site that you can manipulate.

**Next Step:** Since you've mastered the logic of SameSite and SOP, would you like to see a PowerShell script that can automatically detect which of these categories a specific lab's cookies fall into?