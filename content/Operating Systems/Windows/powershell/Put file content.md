```powershell
Set-Content -Path spns.txt -Encoding ASCII -Value @"
IIS_dev/inlanefreight.local:80
"@ 

____

$text = @"
string
@"
Set-Content -Path "cert.txt" -Value $text
```

if the buffer size of the tool you use is smaller than the content size:
```powershell
$text = @"
string
@"

$text += "String"
Set-Content -Path "cert.txt" -Value $text
```