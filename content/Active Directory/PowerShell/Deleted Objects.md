```powershell

Get-ADObject -Filter 'objectClass -eq "user" -and isDeleted -eq $true' -IncludeDeletedObjects

# List all objects and deleted objects
Get-ADObject -Filter * -IncludeDeletedObjects

# List deleted objects only
Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects

# List only deleted users
Get-ADObject -Filter 'objectClass -eq "user"' -IncludeDeletedObjects

# List only deleted computers
Get-ADObject -Filter 'objectClass -eq "computer"' -IncludeDeletedObjects

# Show some useful attributes
Get-ADObject -Filter 'objectClass -eq "user"' -IncludeDeletedObjects |  Select-Object Name, SamAccountName, DistinguishedName, LastKnownParent
```





### restore user.
```
Restore-ADObject -Identity 938182c3-bf0b-410a-9aaa-45c8e1a02ebf
Enable-ADAccount -Identity cert_admin
Set-ADAccountPassword -Identity cert_admin -Reset -NewPassword (ConvertTo-SecureString "Abc123456@" -AsPlainText -Force)
```


```
Get-ADObject -IncludeDeletedObjects -Filter "samAccountName -eq 'theuser'" ` -Properties whenDeleted,msDS-LastKnownParent,msDS-Recycled |  Format-List Name,SamAccountName,DistinguishedName,whenDeleted,msDS-LastKnownParent,msDS-Recycled

Get-ADUser -Filter {SamAccountName -eq 'todd.wolfe'} -Properties Enabled,DistinguishedName |  Format-List SamAccountName,Enabled,DistinguishedName
```

```
Get-ADObject -IncludeDeletedObjects -Filter "samAccountName -eq 'todd.wolfe'" ` -Properties whenDeleted,msDS-LastKnownParent,msDS-Recycled |  Format-List Name,SamAccountName,DistinguishedName,whenDeleted,msDS-LastKnownParent,msDS-Recycled
```