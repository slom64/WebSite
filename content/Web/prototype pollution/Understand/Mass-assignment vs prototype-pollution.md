
> [!question]
> Most prototype pollution are vulnerable to mass assignment in the first place

In many cases, yes—there is a massive overlap. You could think of Prototype Pollution as the **"Ultimate Mass Assignment"** in a JavaScript environment.

Here is a breakdown of how they relate and why they are often the "first place" you find the vulnerability.

---

## 1. The Relationship: Mass Assignment vs. Prototype Pollution

While they look similar, the _impact_ and _target_ are different.
- **Mass Assignment:** Usually targets an **instance** of an object. You add a property like `isAdmin: true` to a specific user object to bypass a check.
- **Prototype Pollution:** Usually targets the **template** (the prototype) of an object. You add `isAdmin: true` to `Object.prototype`, which then grants admin status to **every single object** in the application that doesn't have its own `isAdmin` property.

---
## 2. Why it starts with "Assignment"

For Prototype Pollution to happen, the application must take user-controlled input and **assign** it to an object. This usually occurs in two scenarios:

### A. Recursive Merging (The most common "First Place")

When a developer wants to combine two objects (like a default config and user settings), they often use a "merge" or "extend" function. If that function doesn't check for keys like `__proto__`, it becomes a mass assignment vulnerability that leads to prototype pollution.
```js
// A typical vulnerable "mass assignment" style function
function merge(target, source) {
    for (let key in source) {
        // If 'key' is "__proto__", the code assigns properties to the prototype!
        target[key] = source[key]; 
    }
}
```

### B. Path-based Assignment

Some APIs allow you to set properties via a path, like `user[profile][name]=John`.

If you change this to `user[__proto__][isAdmin]=true`, you aren't just assigning a value to the "user"; you are exploiting the mass-assignment logic to reach the global prototype.

---

## 3. Investigating "Assignment" via PowerShell

In the PortSwigger labs, you are often looking for where the "assignment" happens in the JavaScript files. You can use PowerShell to find these "Mass Assignment" code patterns in the site's source code:

### Scan for Property Assignment Patterns

This command searches for dynamic property assignments (bracket notation) which are the most common "sinks" for this vulnerability.
```powershell
# Search for patterns where an object is being assigned a value via a variable key
# This often indicates a potential mass assignment sink.
Get-ChildItem -Recurse -Filter "*.js" | Select-String -Pattern '\[.*\]\s*='
```
### Identify Merging Libraries

Many prototype pollution bugs come from old versions of popular libraries that perform mass assignments (like `lodash` or `jquery`). You can check the versions via PowerShell:
```powershell
# Check for common vulnerable libraries in your local project or downloaded JS
Get-ChildItem -Recurse -Filter "package.json" | Select-String -Pattern "lodash|jquery|merge"
```

---

## Summary Table

|**Feature**|**Mass Assignment**|**Prototype Pollution**|
|---|---|---|
|**Primary Goal**|Modify one specific object.|Modify the base template for all objects.|
|**Source**|Unfiltered user input in request body/URL.|Unfiltered user input in request body/URL.|
|**Logic**|`obj.isAdmin = true`|`obj.__proto__.isAdmin = true`|
|**Language**|Most languages (Java, C#, Ruby, JS).|**Strictly JavaScript.**|
