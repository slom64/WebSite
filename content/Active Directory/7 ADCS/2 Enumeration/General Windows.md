
Checking who is authorized to publish certificates.
```powershell
PS C:\Tools> net localgroup "Cert Publishers"
Alias name     Cert Publishers
Comment        Members of this group are permitted to publish certificates to the directory

Members

-------------------------------------------------------------------------------
LAB-DC$
```

```powershell
PS C:\Tools> .\Certify.exe find
```