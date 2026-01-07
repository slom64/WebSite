

```html
GET /post?postId=6&SLOM=SLOM


____RESPONSE

<iframe onload='this.height = this.contentWindow.document.body.scrollHeight + "px"' src='/post/comment/comment-form#SLOM=SLOM&postId=6'></iframe>
```

```html
/post/comment/comment-form?#SLOM=SLOM&postId=6
_________RESPONSE

<script>
	parent.postMessage({type: 'onload', data: window.location.href}, '*')
	function submitForm(form, ev) {
		ev.preventDefault();
		const formData = new FormData(document.getElementById("comment-form")); // Get form data.
		const hashParams = new URLSearchParams(window.location.hash.substr(1)); // Get fragment data.
		const o = {};
		formData.forEach((v, k) => o[k] = v); // put form data in o['name'] = 'abc'.
		hashParams.forEach((v, k) => o[k] = v); // put fragment data in o['SLOM'] = 'SLOM'.
		parent.postMessage({type: 'oncomment', content: o}, '*');
		form.reset();
	}
</script>
```

```
mutationType {name}
subscriptionType {name}
types {...FullType}
directives {name 
description
args {...InputValue}
```