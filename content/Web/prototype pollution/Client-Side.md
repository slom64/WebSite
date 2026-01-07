- You mainly add `__proto__.canary = "random"` and enumerate `Object` or other objects in DOM if they got `canary` property.
- **Prototype pollution via the constructor**: some times `__proto__` is heavily sanitized, so we need another way to trigger prototype pollution
	- `myObject.constructor.prototype.abc=abc`
	- `__pro__proto__to__.gadget=payload`
	- `#__proto__[testproperty]=DOM_INVADER_PP_POC`
- **Prototype pollution in external libraries**:



## Finding client-side prototype pollution sources manually

Finding [prototype pollution sources](https://portswigger.net/web-security/prototype-pollution#prototype-pollution-sources) manually is largely a case of trial and error. In short, you need to try different ways of adding an arbitrary property to `Object.prototype` until you find a source that works.

When testing for client-side vulnerabilities, this involves the following high-level steps:

1. Try to inject an arbitrary property via the query string, URL fragment, and any JSON input. For example:
    `vulnerable-website.com/?__proto__[foo]=bar`
2. In your browser console, inspect `Object.prototype` to see if you have successfully polluted it with your arbitrary property:
    `Object.prototype.foo // "bar" indicates that you have successfully polluted the prototype // undefined indicates that the attack was not successful`
3. If the property was not added to the prototype, try using different techniques, such as switching to dot notation rather than bracket notation, or vice versa:
    `vulnerable-website.com/?__proto__.foo=bar`
4. Repeat this process for each potential source.

Try this in source:
```
/?__proto__[abc]=abc&constructor.prototype.abc=abc&__prot__proto__o__[abc]=abc#__proto__[abc]=abc&constructor.prototype.abc=abc&__prot__proto__o__[abc]=abc
```

Then check in console:
```js
// This code have better feature than DOM invader because it checks all objects, if one object has __proto__.canary = random then that mean we have hit.
function detectPrototypePollution(canary = "polluted") {
    console.log(`[*] Scanning for prototype pollution with canary: "${canary}"`);
    
    const found = [];
    const visited = new WeakSet();
    
    function checkPrototypeChain(obj, path = "unknown") {
        if (!obj || visited.has(obj)) return;
        
        try {
            visited.add(obj);
            
            // Walk up the prototype chain
            let proto = Object.getPrototypeOf(obj);
            let level = 0;
            
            while (proto !== null) {
                // Check if this prototype level has the canary as OWN property
                const ownProps = Object.getOwnPropertyNames(proto);
                
                if (ownProps.includes(canary)) {
                    const location = proto === Object.prototype 
                        ? "Object.prototype (ROOT)" 
                        : `Intermediate prototype (level ${level})`;
                    
                    found.push({
                        path: path,
                        location: location,
                        prototype: proto,
                        constructor: proto.constructor?.name || "Unknown"
                    });
                    
                    console.warn(`[!] POLLUTION DETECTED!`);
                    console.log(`    Path: ${path}`);
                    console.log(`    Level: ${location}`);
                    console.log(`    Constructor: ${proto.constructor?.name || "Unknown"}`);
                    console.log(`    Value: ${proto[canary]}`);
                }
                
                proto = Object.getPrototypeOf(proto);
                level++;
            }
        } catch (e) {
            // Skip inaccessible objects
        }
    }
    
    function scanObject(obj, path, depth = 0) {
        if (depth > 5) return; // Limit recursion depth
        if (!obj || typeof obj !== 'object') return;
        if (visited.has(obj)) return;
        
        try {
            visited.add(obj);
            checkPrototypeChain(obj, path);
            
            // Recursively check properties
            const props = Object.getOwnPropertyNames(obj);
            for (const prop of props) {
                try {
                    const value = obj[prop];
                    if (value && typeof value === 'object') {
                        scanObject(value, `${path}.${prop}`, depth + 1);
                    }
                } catch (e) {
                    // Skip inaccessible properties
                }
            }
        } catch (e) {
            // Skip
        }
    }
    
    // Scan all window properties
    const windowProps = Object.getOwnPropertyNames(window);
    for (const prop of windowProps) {
        try {
            const value = window[prop];
            if (value && typeof value === 'object') {
                scanObject(value, `window.${prop}`, 0);
            }
        } catch (e) {
            // Skip inaccessible window properties
        }
    }
    
    // Always check common prototypes directly
    const commonPrototypes = [
        { proto: Object.prototype, name: "Object.prototype" },
        { proto: Array.prototype, name: "Array.prototype" },
        { proto: String.prototype, name: "String.prototype" },
        { proto: Function.prototype, name: "Function.prototype" }
    ];
    
    for (const {proto, name} of commonPrototypes) {
        if (Object.prototype.hasOwnProperty.call(proto, canary)) {
            console.warn(`[!] DIRECT POLLUTION on ${name}!`);
            console.log(`    Value: ${proto[canary]}`);
            found.push({ path: name, location: "Direct", prototype: proto });
        }
    }
    
    if (found.length === 0) {
        console.log(`[-] No pollution detected with canary "${canary}"`);
    } else {
        console.log(`[+] Found ${found.length} polluted prototype(s)`);
    }
    
    return found;
}

// Usage:
detectPrototypePollution("abc");
```

Exploit:
- Finding sink is very time consuming because you need to go throw javascript code to explore what things developers have forgot to put a default value for.
- So use DOM invader to explore `sinks`.

```js
// You can't just do the following, because the main idea of prototype pollution is to inject property in parent object using functions the indirectly do this in string value.
__proto__[toString]=alert(1) // this mean Object.prototype.toString = "alert(1)". DOESN'T EXECUTE COMMAND, treat function as argument string.

```