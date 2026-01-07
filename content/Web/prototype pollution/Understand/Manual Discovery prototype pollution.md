
> [!Question] 
> to discover prototype pollution in client side "URL source". we will put `?__proto__.random = random2`. then we will go throw every object in DOM and if we found object that has random. that means we have prototype pollution and depends on how many parent prototypes it has we will repeat `__proto__ `many times until we reach Object prototype

You’ve got the core logic perfectly right: you are injecting a "canary" (the random string) and then checking if that canary has "leaked" into the global object template.

However, there is one technical nuance in your plan regarding the "many times" part that we should clarify to save you time during your labs.

### 1. The "Object.prototype" Shortcut
In JavaScript, you almost never need to repeat `__proto__` many times in a URL to find the "root."
Nearly every object you encounter in a browser (a `div`, an `array`, a `string`) eventually leads back to the same single source: **`Object.prototype`**.
- If you pollute `Object.prototype`, you have already reached the top.
- Any object created after that pollution will inherit your "random" property, regardless of how deep its prototype chain is.

---
### 2. The Discovery Workflow (Step-by-Step)

When you are testing a URL source in PortSwigger Academy, your manual "discovery" flow looks like this:
1. Inject the Canary:
    https://vulnerable-site.com/?__proto__[testpolluted]=joker
2. Check the Global Template:
    Open the Browser Console (F12) and type:
    Object.prototype.testpolluted
3. **The Result:**
    - If it returns `"joker"`, you have **Global Prototype Pollution**.
    - If it returns `undefined`, the application is either not vulnerable or is using a "sanitized" merge logic.
---

### 3. Automating Discovery with PowerShell

Since you are a PowerShell user, you can actually automate the "crawling" of a site's JavaScript files to look for the **Sinks** (the code that actually performs the assignment) before you even try the URL payload.

This script searches for the `location.search` or `location.hash` sources being passed into dangerous functions:

```js
# Define the directory where you've saved the site's JS files
$jsPath = "C:\Path\To\Downloaded\JS"

# Search for "Sources" (URL data) meeting "Sinks" (Assignment logic)
Get-ChildItem -Path $jsPath -Recurse -Filter "*.js" | ForEach-Object {
    $content = Get-Content $_.FullName
    if ($content -match "location\.search|location\.hash|location\.href") {
        Write-Host "Potential SOURCE found in: $($_.Name)" -ForegroundColor Cyan
        # Look for bracket notation assignment nearby
        $content | Select-String -Pattern "\[.*\]\s*=" | Write-Host -ForegroundColor Yellow
    }
}
```

---

### 4. When would you use "nested" `__proto__`?

You mentioned repeating `__proto__`. This is usually only necessary if the developer has tried to implement a **flawed defense**.

Some developers write code like this:

if (key === '`__proto__`') continue;

To bypass this, hackers sometimes use:
- `constructor[prototype][testpolluted]=joker` (Accessing the prototype via the constructor instead).
- Nested objects if the merge is deep, but usually, `__proto__` is a direct "teleport" to the root.

### Summary Checklist for your Lab:
1. **Find a Source:** Look for code that reads from the URL (`searchParams`).
2. **Find a Sink:** Look for code that merges that data into an object.
3. **Pollute:** Use `?__proto__[yourProperty]=yourValue`.
4. **Verify:** Check `Object.prototype.yourProperty` in the console.
5. **Exploit:** Find a "Gadget" (code that uses that property) to trigger XSS or bypass logic.
