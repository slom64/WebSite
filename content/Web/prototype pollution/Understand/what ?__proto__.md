
> [!question] 
 i want to understand something, 
when we do `asdf.__proto__` where are getting the parent prototype of object asdf.
but sometimes i see `http://domain.com?__proto__`
now that `__proto__` refer to who and what object

In the context of Prototype Pollution, understanding the difference between a **JavaScript property access** and a **URL parameter** is the key to mastering this vulnerability.

When you see `?__proto__[isAdmin]=true` in a URL, it is not an object itself; it is a **payload** waiting to be processed by a vulnerable script on the server or in the browser.

---

## 1. The "Who" and "What" of `?__proto__`

In a URL, `__proto__` is just a string. It doesn't refer to any object until a **sink** (a piece of code) processes it.

### The Target Object

Usually, the target is a **global or base configuration object** used by the application. Most JavaScript objects eventually inherit from `Object.prototype`. If you can "pollute" `Object.prototype`, you affect almost every object in the runtime environment.

### The Mechanism

The pollution happens when the application takes that URL string and merges it into an existing object using a recursive merge or clone function.

---

## 2. How it works in Practice

### The Vulnerable Code Pattern

Imagine the website has a script that looks like this:
```js
// A common utility function to merge settings
function merge(target, source) {
    for (let key in source) {
        if (typeof target[key] === 'object' && typeof source[key] === 'object') {
            merge(target[key], source[key]);
        } else {
            target[key] = source[key];
        }
    }
}

// 1. The URL ?__proto__[layout]=dark is parsed into an object:
// let queryParams = { "__proto__": { "layout": "dark" } };

// 2. The app merges it into a local config:
let config = { version: "1.0" };
merge(config, queryParams);
```

### What happens here?

When the `merge` function hits the key `__proto__`, it doesn't just create a new property called "proto". Because `__proto__` is a special accessor, `target["__proto__"]` actually points to **`Object.prototype`**.

The function then sets `Object.prototype.layout = "dark"`. Now, **every single object** in the application has a `layout` property set to "dark" by default.

---
## 4. Key Takeaway

- **`asdf.__proto__`**: You are actively traversing the chain of a specific instance.
- **`?__proto__`**: You are sending a "key name" to the application. If the application is "gadget-blind" and uses that key to perform an assignment like `obj[key] = value`, it accidentally overwrites the root prototype of the entire engine.

Would you like me to provide a PowerShell script to help you automate the discovery of hidden parameters (parameter discovery) on a target URL?