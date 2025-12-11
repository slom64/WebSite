Displays or modifies discretionary access control lists (DACLs) on specified files and applies stored DACLs to files in specified directories.

```powershell
PS C:\htb> icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F

processed file: C:\Department Shares\Private\IT\cred.txt
Successfully processed 1 files; Failed processing 0 files
```
