Mapping
Think of ESC10 as _“ESC9 but enforced globally by registry misconfig instead of per-template misconfig.”_

---
## 🧩 Case 1 — **Kerberos side (StrongCertificateBindingEnforcement "SCBE" = 0)**

### 1. What’s supposed to happen
- Since ESC6, Microsoft introduced **`StrongCertificateBindingEnforcement` (SCBE)** to prevent blind UPN trust during Kerberos logon.
- Normally (SCBE = 1 or 2):
    - The KDC binds the cert to the user by SID, not by the UPN you typed into the CSR.
    - That means if you request `UPN=administrator@corp.local`, the CA overrides it or the KDC ignores it.
### 2. Misconfiguration
- If SCBE = `0`, it means: **“do legacy weak binding”** → the KDC **doesn’t enforce SID binding**, and falls back to trusting UPN directly.
- This reopens ESC6 globally, regardless of templates.
### 3. Abuse chain
1. Find any template that issues client authentication certs (like built-in `User` template).
2. Attacker requests a cert but specifies **UPN = administrator@corp.local**in the CSR.
3. CA issues it (because legacy behavior).
4. KDC accepts the cert as Administrator (because SCBE=0 = no strong binding).

🚨 Same outcome as ESC9, but the root cause is registry misconfig, not template flag.

---
## 🧩 Case 2 — **Schannel side (CertificateMappingMethods = 0x4)**

### 1. What Schannel does
- Schannel = Microsoft’s TLS/SSL provider.
- When you authenticate to something like RDP, LDAPS, or IIS with a certificate, **Schannel maps the cert to an AD account**.
- Normally, Schannel should map the cert securely (by SID, explicit mapping, etc.).
### 2. Misconfiguration
- `CertificateMappingMethods` defines how mapping is done.
- Value `0x4` = “UPN mapping only, no strong checks.”
- So Schannel will **blindly trust the UPN in the certificate** to map you to an account.
### 3. Abuse chain
1. Attacker finds a template with client authentication EKU.
2. Attacker requests a cert with `UPN=administrator@corp.local`.
3. CA issues cert.
4. Attacker uses this cert over TLS (RDP/LDAPS/IIS, etc.).
5. Schannel maps them directly to Administrator, no SID validation.

🚨 Again, attacker logs in as someone else — but this time via TLS-based authentication instead of Kerberos.

---
## 🔹 ESC10 vs ESC9 (the contrast)
- **ESC9** → Template misconfig (`CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`) lets you supply UPN.
- **ESC10** → Registry misconfig (`SCBE=0` or `CertificateMappingMethods=0x4`) means system-wide fallback to weak UPN mapping.
So in both cases, attacker controls UPN → impersonates privileged users.

---
## 🔹 Abuse Requirements Clarified
### Case 1 (Kerberos SCBE=0)
- Need a template with ClientAuth EKU (so you can logon with it).
- Need to be able to enroll in it.
- If you can also modify another account (`GenericWrite` on B), you could e.g., set its UPN to what you want and then logon as B.
### Case 2 (Schannel Mapping=0x4)
- Same as above, but applied to TLS mapping.
- Caveat: if target account already has a UPN, you may hit constraint errors → easier to abuse with machine accounts or Administrator accounts that don’t have UPN.
---
## 🔹 Summary

- ESC10 is **misconfigured registry keys** that **weaken certificate mapping**.
- Two flavors:
    - **Kerberos (SCBE=0)** → like ESC6 all over again.
    - **Schannel (Mapping=0x4)** → blind UPN trust in TLS authentication.
- Result = **attacker-supplied UPN in cert → privilege escalation**.

---

> [!NOTE] 
> low-privileged user doesn't have the right to read the registry key's values. So, we need to try the attack to identify if it is vulnerable or not. "blind testing --guessing"
