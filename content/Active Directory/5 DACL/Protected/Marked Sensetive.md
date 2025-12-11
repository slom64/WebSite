
In bloodhound If this set to false that mean we can do delegation to this user. 
![[Z Assets/Images/Pasted image 20251113083405.jpeg]]

But in powershell, with `AccountNotDelegated` = false. if it was True then we can't use delegation to impersionate this user.
```powershell
Get-ADUser backupAdmin -Properties AccountNotDelegated

AccountNotDelegated : False
DistinguishedName   : CN=backupadmin,CN=Users,DC=rustykey,DC=htb
Enabled             : True
GivenName           : backupadmin
Name                : backupadmin
ObjectClass         : user
ObjectGUID          : adea093f-5ec8-4ac7-9da3-196a1b1fa37d
SamAccountName      : backupadmin
SID                 : S-1-5-21-3316070415-896458127-4139322052-3601
Surname             : 
UserPrincipalName   : backupadmin@rustykey.htb
```