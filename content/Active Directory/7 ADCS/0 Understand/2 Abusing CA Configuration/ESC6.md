
# ESC6 — The “SAN Injection” Problem

### 1. What the flag does

- The **`EDITF_ATTRIBUTESUBJECTALTNAME2`** flag on the CA means:    
    - _“Trust whatever Subject Alternative Name (SAN) the requester includes in their CSR.”_
- In other words, if the CSR says:
```
 SAN: UPN=administrator@corp.local
```
the CA will happily issue that certificate with that SAN.

---
### 2. Why that’s dangerous
- Remember: **SAN → UPN** is how **Kerberos maps certificates to AD accounts**.
- So if the CA lets you pick your own SAN/UPN, you can just say:
    - “I’m `administrator@corp.local`” in the CSR.
- When you use that cert for SmartCard/PKINIT logon:
    - KDC looks at SAN → sees `administrator@corp.local` → maps it → issues TGT as Administrator.
- 🚨 Boom → escalation to DA.
---
### 3. Relation to ESC1
- ESC1 is about **enrollment agents** being able to request certs on behalf of others.
- With ESC6 + SAN misconfig, you **don’t even need enrollment agent**.
- Any vulnerable template (even `User`) becomes abusable because SAN is attacker-controlled.
---
### 4. The Patch (CVE-2022-26923 / Certifried)
Microsoft fixed this in **May 2022** by:
- Adding the `szOID_NTDS_CA_SECURITY_EXT` (with the requester’s **objectSID**) into issued certificates.
- Updating **KDC mapping rules** (`StrongCertificateBindingEnforcement`).

So after patch:
- Even if cert has `SAN=administrator@corp.local`, the KDC will check:
    - Does this cert’s `objectSid` = Administrator’s SID?
    - ❌ No → authentication denied.        
---
### 5. Why it’s still relevant
- Many orgs **did not patch** or still allow `EDITF_ATTRIBUTESUBJECTALTNAME2`.
- In that case:
    - Every template that allows SAN supply → becomes privilege escalation vector.
    - Built-in **User** template is often enough.
---
# 🔹 Summary of ESC6

- Misconfig: `EDITF_ATTRIBUTESUBJECTALTNAME2` flag = users control SAN.
- Effect: You can set `SAN=UPN=administrator@corp.local` → logon as Administrator.
- Patch: CVE-2022-26923 → certs now bind to **SID**, not just UPN.
- Today: Only exploitable if CA unpatched + SAN supply allowed.
---

👉 Quick analogy:

- **ESC6 = “CA lets you write your own passport name.”**    
- **ESC9 = “CA lets you edit your passport name because of template misconfig.”**
- **ESC10 = “Border control ignores your photo/signature and just trusts the name on the passport.”**

---

Do you want me to **show the exact difference in where trust fails** (ESC6 vs ESC9 vs ESC10) with a comparison flow diagram, so you can see “who trusted the UPN” in each case?