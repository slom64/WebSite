## What is XML?
XML stands for "extensible markup language". XML is a language designed for storing and transporting data. Like HTML, XML uses a tree-like structure of tags and data. Unlike HTML, XML does not use predefined tags, and so tags can be given names that describe the data. Earlier in the web's history, XML was in vogue as a data transport format (the "X" in "AJAX" stands for "XML"). But its popularity has now declined in favor of the JSON format.

### What are XML entities?
XML entities are a way of representing an item of data within an XML document, instead of using the data itself. Various entities are built in to the specification of the XML language. For example, the entities `&lt;` and `&gt;` represent the characters `<` and `>`. These are metacharacters used to denote XML tags, and so must generally be represented using their entities when they appear within data.

### What is document type definition?
The XML document type definition (DTD) contains declarations that can define the structure of an XML document, the types of data values it can contain, and other items. The DTD is declared within the optional `DOCTYPE` element at the start of the XML document. The DTD can be fully self-contained within the document itself (known as an "internal DTD") or can be loaded from elsewhere (known as an "external DTD") or can be hybrid of the two.
```xml
<?xml version="1.0" encoding="UTF-8"?>
------------------------------------------
<!DOCTYPE foo [ 
	<!ENTITY myentity "my entity value" > 
]>
------------------------------------------
```

---
## There are 3 types of Entities
- **Custom Entities**
- **External Entities**
- **Parameter Entities**

---
### What are XML `custom entities`?
XML allows custom entities to be defined within the DTD. For example:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ 
	<!ENTITY myentity "my entity value" > 
]>
<stockCheck><productId>&myentity;</productId></stockCheck>
```
This definition means that any usage of the entity reference `&myentity;` within the XML document will be replaced with the defined value: "`my entity value`".

---
### What are XML `external entities`?
XML external entities are a type of custom entity whose definition is located outside of the DTD where they are declared.

The declaration of an external entity uses the `SYSTEM` keyword and must specify a URL from which the value of the entity should be loaded. For example:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ 
	<!ENTITY ext SYSTEM "http://normal-website.com" > OR
	<!ENTITY ext SYSTEM "file:///etc/passwd" > 
]>
<stockCheck><productId>&ext;</productId></stockCheck>
```
The URL can use the `file://` protocol, and so external entities can be loaded from file. For example:

---
### What a `parameter entity` is
- **Parameter entities** are entities that are _defined and expanded inside the DTD only_.
- Syntax for definition: `<!ENTITY % name "replacement-text">` (note the `%` after `ENTITY`).
- To use/expand them inside the DTD you write: `%name;` (note the `%` when referencing).
- They are **not** referenced in the XML document body — only inside the DTD. General entities (the ones you already know) are referenced in content with `&name;`.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ 
	<!ENTITY % ext SYSTEM "http://normal-website.com" >
	%ext; 
]>
```

#### How parameter differ from general entities
- **General entity** (usable in XML content):  
    `<!ENTITY foo "bar">` → referenced in the XML as `&foo;`
- **Parameter entity** (usable only in DTD):  
    `<!ENTITY % pfoo "SOME DTD TEXT">` → referenced in the DTD as `%pfoo;`