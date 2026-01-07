
> [!Question ] 
> How Authorization server protect redirect_uri? The attack can put his url and then get the `code` then access other people resources. 


> [!Summary] 
> When user gets `code` from authorizatoin server, this code is tied with `redirect_uri`. And when client try to use the `code` to get `access token` it submit hardcoded `redirect_uri` which create a mismatch in the authorization server because the code have been issued to different `redirect_uri`so rejection happens.
> 
> And the attacker can't just behave as client and send `code` to authorization server to get `access token`, because attacker doesn't know the client `secret_key`. So the attacker should feed the `code` to client, then client will do the process of getting `access token`. If authorization server is misconfigured, it will accept the `code` and give back access token, so at the end the attakcer get access to resources of the victim user.


To understand how the Authorization Server (AS) uses the `redirect_uri` to stop an attacker, you have to look at the flow as a **two-step puzzle** where the pieces must match exactly.
### The Logic of the Fix

The vulnerability exists because, in Step 1, an attacker might trick the AS into sending the `code` to a "leaky" or malicious URI. The "fix" is to force the Client to prove they know where that code was sent when they come back to trade it for a token.

Here is the step-by-step logic of how the Authorization Server validates this:

---

### Step 1: The Request (The Setup)

The attacker crafts a link with a modified `redirect_uri` that they control:
```http
GET /authorize?client_id=123&redirect_uri=https://attacker.com/callback&response_type=code
```

If the AS is misconfigured (e.g., it only checks that the domain starts with `yelp.com` or doesn't check at all), it generates a `code` and sends it to `attacker.com`. **The AS now "remembers" that Code A was issued for Redirect_URI X.**

### Step 2: The Exchange (The Validation)

Now, the attacker has the code. They want to use it to log into the real `yelp.com` as the victim. They send the code to the real Yelp backend (or try to use it themselves).

To get the token, a request must be made to the `/token` endpoint. The AS **requires** the `redirect_uri` to be sent again in this POST request:
```http
POST /token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=Code_A&
client_id=123&
client_secret=secret456&
redirect_uri=https://yelp.com/callback  <-- This is what the LEGIT client will send
```

### Step 3: The "Matching" Check

When the Authorization Server receives this POST request, it does a 3-way check:
1. Is the `code` valid? (Yes)
2. Is the `client_secret` correct? (Yes)
3. **Does the `redirect_uri` in this POST request match the `redirect_uri` that was used when this specific `code` was created?**

**This is the failure point for the attacker:**
- The AS sees that **Code A** was originally sent to `https://attacker.com/callback`.
- But the current request claims the redirect was `https://yelp.com/callback`.
- **Result:** **Mismatch.** The AS rejects the request and refuses to issue an Access Token.
---
### Why the attacker can't just lie in Step 2?

You might ask: _"Why doesn't the attacker just send `https://attacker.com/callback` in the POST request to make it match?"_
1. **Confidential Clients:** If Yelp is a web app, the `/token` exchange happens **server-to-server**. The attacker doesn't have the `client_secret`, so they can't make this request themselves. They have to "feed" the code to Yelp's backend.
2. **Yelp's Backend is Hardcoded:** Yelp’s actual backend code is hardcoded to send its own real URL (`https://yelp.com/callback`) in that POST request. It doesn't know (and doesn't care) what the attacker put in the browser URL; it always identifies itself with its true callback.

### Summary Table: The Defense

| **Action**          | **Attacker's Goal**           | **The AS Response (Secure)**                               |
| ------------------- | ----------------------------- | ---------------------------------------------------------- |
| **Initial Request** | Send code to `evil.com`       | AS notes: `Code_XYZ` belongs to `evil.com`.                |
| **Token Exchange**  | Use `Code_XYZ` at `legit.com` | Legit server sends `redirect_uri=legit.com`.               |
| **The Comparison**  | Bypass security               | AS sees `evil.com != legit.com` and **Kills the session**. |
