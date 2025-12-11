There is special flag in`msPKI-Enrollment-Flag` which is **`CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`** (value: `0x00000001`), this flag grante the user the rights to change the UPN and the CA won't check the SID which mean it disable the `szOID_NTDS_CA_SECURITY_EXT` extension.

---

> [!question]
> cool now lets start with ESC9. it lavarege the flaw of msPKI-Enrollment-Flag which override StrongCertificateBindingEnforcement so we can change the UPN and the CA will take in consideration the request UPN

Exactly 👍 you’ve got the right starting point for **ESC9**. Let’s break it down carefully, because ESC9 is all about how **StrongCertificateBindingEnforcement (SCBE)** was supposed to close ESC6, but there’s a template flag (`msPKI-Enrollment-Flag`) that can undercut it.

---
## 🔹 Recap: What SCBE does

- After **Certifried (ESC6)**, Microsoft patched AD CS so that the **KDC no longer blindly trusts the UPN** in a certificate.    
- Instead, it looks at the **`szOID_NTDS_CA_SECURITY_EXT` extension** (with the requester’s `objectSid`) that the CA injects.
- If `SCBE` `"StrongCertificateBindingEnforcement"` is set to `1` (compatibility) or `2` (full enforcement), the KDC will **bind the cert to the account by SID**, not by UPN, unless there’s an explicit mapping.
- This closes the “I request a cert with UPN=Administrator” trick. ✅

---
## 🔹 Where `msPKI-Enrollment-Flag` comes in

Certificate templates in AD have a property: **`msPKI-Enrollment-Flag`**.  
One of its bits is:
- **`CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`** (value: `0x00000001`).

👉 If this flag is set, the **enrollee (the requester)** is allowed to **supply their own Subject / SubjectAltName (including UPN, DNS, etc.)** in the CSR.

So if the CA uses this template:
- It does **not override the CSR UPN with the actual AD account’s UPN**.
- It **honors the user-supplied SubjectAltName** instead.
- The result: The issued certificate contains whatever UPN you asked for, even if it’s not yours. 🚨

That’s how ESC9 **reopens the ESC6 hole**, but now it’s template-driven instead of global.

---

## 🔹 ESC9 Abuse Flow (step-by-step)

1. Attacker is an authenticated AD user (low-priv).    
2. Attacker finds a vulnerable template with:
    - `Client Authentication` EKU (so it can be used for logon/auth).
    - `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` set in `msPKI-Enrollment-Flag`.
    - Enrollment rights granted to them.
3. Attacker generates CSR with **SubjectAltName:UPN=[administrator@corp.local](mailto:administrator@corp.local)**.
4. Attacker submits CSR to CA.
5. CA issues certificate with that UPN, because the template says “enrollee supplies subject.”
6. Attacker now has a cert that the KDC will accept for logon as Administrator.

Even if SCBE is `1` or `2`, the KDC sees:
- Certificate is valid, signed by trusted CA.
- It has a UPN = `administrator@corp.local`.
- Because the CA itself stamped it that way (following the template rules), the KDC accepts it.

---
## 🔹 Why is this possible?

Because **Microsoft’s patch assumed the CA would always bind the cert to the authenticated requester’s identity**.  
But if the template says “nah, trust whatever the user asks for,” the CA happily signs attacker-controlled UPNs.

So:
- ESC6 = flaw in KDC trusting cert UPN blindly.
- ESC9 = flaw in CA template allowing attacker to _control_ what UPN the CA stamps, bypassing the SID extension idea.
---
## 🔹 Summary

- **msPKI-Enrollment-Flag** controls how much of the Subject is requester-controlled.
- If `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` is set, attacker can specify arbitrary UPN in CSR.
- Even with `StrongCertificateBindingEnforcement` on, the KDC trusts it because the CA (a trusted authority) signed it.
- This is **ESC9**: Template misconfiguration reintroduces the “I can impersonate anyone by UPN” problem.

---
# ESC9 Abuse Requirements

To successfully abuse this misconfiguration, specific prerequisites must be met:
1. The `StrongCertificateBindingEnforcement` registry key should not be set to 2 (by default, it is set to 1 ), or the `CertificateMappingMethods` should contain the UPN flag ( 0x4 ). Regrettably, as a low-privileged user, accessing and reading the values of these registry keys is typically unattainable.
2. The certificate template must incorporate the `CT_FLAG_NO_SECURITY_EXTENSION` flag within the msPKI-Enrollment-Flag value.
3. The certificate template should explicitly specify client authentication as its purpose.
4. The attacker must possess at least the GenericWrite privilege against any user account (account A) to compromise the security of any other user account (account B).