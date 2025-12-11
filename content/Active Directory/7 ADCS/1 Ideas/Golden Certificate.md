# Idea

If you were able to get the `private key` of the CA then you can generate any certificate as you like.

The private key is in `CORP-CA.pfx` and we will create certificate that is for the administrator:
```
> certipy forge \
    -ca-pfx 'CORP-CA.pfx' -upn 'administrator@corp.local' \
    -sid 'S-1-5-21-...-500' -crl 'ldap:///'    

[*] Saving forged certificate and private key to 'administrator_forged.pfx'
[*] Wrote forged certificate and private key to 'administrator_forged.pfx'

> certipy auth -pfx 'administrator_forged.pfx' -dc-ip '10.0.0.100'
```

---
# How to get private key of CA? 

## CA certificate

### 🔑 What is a CA certificate?

- A **CA certificate** is a certificate where the `Basic Constraints` extension has `CA=TRUE`.    
- ==Its private key can be used for **signing other certificates** (issuing them), not just authenticating itself.==
- It’s the “root of trust” in a PKI hierarchy: all other certs (users, machines, services) chain back up to one or more CA certs.
Think of a CA cert as the **master signing authority** in that environment:
- **Root CA certificate** = the absolute trust anchor. Self-signed (issuer == subject).
- **Subordinate CA (Issuing CA) certificate** = signed by another CA, delegated to issue certs for users/machines.
---
## ⚙️ How CA certificates are different from “normal” certs
1. **Key Usage**
    - End-entity certs: usually limited to `Digital Signature`, `Key Encipherment`, etc.
    - CA certs: must include `keyCertSign` and `cRLSign`. Without those, they can’t sign other certs.
2. **Basic Constraints**
    - End-entity certs: `CA=FALSE`.
    - CA certs: `CA=TRUE`. May also include a `pathlen` constraint (limiting how many subordinate CAs can be issued below it).
3. **Template / issuance**
    - In Active Directory Certificate Services (AD CS), CA certificates are not usually requested like normal user/machine certs. They’re created during **CA installation**.
    - There is a “CA” template (like in your output: `Template: CA, Root Certification Authority`) but it’s only usable for CA setup, not for normal users.
4. **Storage & protection**
    - Root CA private keys are usually offline (protected in HSM or at least not online all the time).
    - Issuing CA private keys are online, but ideally HSM-protected, with strict ACLs.
    - Normal user/machine cert private keys are just in software KSP by default.
---
## 🧾 Can you request a CA certificate normally?
No — not in a healthy environment:
- **Normal users** cannot request CA certificates. The CA template isn’t published for them to enroll against.
- CA certificates are created when setting up an AD CS role (Enterprise CA, Standalone CA, Subordinate CA).
- Admins may import/export CA certs and private keys (e.g., for backup/restore or moving to another host).

If you as a normal user tried to “request” a CA cert via `certreq`, the CA wouldn’t allow it unless it’s misconfigured (like if someone accidentally published a template with `CA=TRUE` and gave Enroll rights to users — that would be catastrophic).

---
## 🛑 Why are CA certificates so sensitive?
- Whoever controls a CA’s private key can issue new certs for _any identity_ in the forest/domain.    
- With that, they can mint a certificate for `Administrator@domain.local`, then use it for authentication (Kerberos PKINIT or smart card logon).
- This is exactly what makes attacks like **Golden Certificate** or **ESC1/ESC2/ESC4 in ADCS exploitation** so devastating.
---
## 🧭 How they show up in `certutil -store`

On a CA host, you’ll see:

```
Subject: CN=Certificate-LTD-CA, DC=certificate, DC=htb
Template: CA, Root Certification Authority
Basic Constraints: Subject Type=CA
Key Usage: KeyCertSign, CRLSign
```

That tells you:  
✔ It’s a CA certificate  
✔ The private key on this machine is the **signing key** for the CA.

On a normal workstation, you’ll also see CA certs (in `Root` or `CA` store), but they are **public only** — no private key is present.

---

## 📦 Import/export of CA certs

- The **certificate** (public) can be freely imported/exported (it has to be distributed so others can trust it). 
	- Thats why you may find the private key exportable if you can access the CA, which means you have the public key and you may have the private key.
- The **private key** should _only_ exist on the CA (or backup media, or HSM).
- If you find it on a workstation (exportable in PFX), that means:
    - The CA key was imported there (bad practice), or
    - The workstation _is_ a CA (Enterprise CA role installed), or
    - A misconfigured admin restored/loaded the CA key material.
---

✅ **Summary**
- CA certs = special type with `CA=TRUE`, `keyCertSign`, `cRLSign`. 
- They’re created during CA setup, not requested by normal users.
- Public CA certs are everywhere (in Trusted Root/Intermediate stores).
- The private key of a CA cert should _only_ exist on the CA itself (or HSM).
- If you can export a CA cert + private key from any host, you own the PKI: you can mint certs for anyone.
---

`certutil -store`: shows us the certificate that is stored right now. and the private key that is inside the certificate if presents.

> [!question] 
> cool, now when i do certutil -store and there is private key can be exported, this private key is the local private key that is in relation with the public key in the certificate or is the private key used to sign the certificate?

Short answer: **The private key shown by `certutil -store` is the private key that _pairs with the public key in that certificate_.**  
Whether that private key is _a CA signing key_ or _an end-entity key_ depends entirely on what the certificate represents (its Key Usage / Basic Constraints / Template).

Below is a clear breakdown plus exact signs and commands you can use to tell which is which.

---
### 1) Relationship: certificate ↔ private key
- Every certificate contains a **public key**. 
- If `certutil -store` reports **“Private key is present”**, that means the matching **private key** for _that certificate’s public key_ exists on that machine.
- **So the private key is always "in relation with the public key in the certificate."**
    - If the certificate is an **end-entity cert** (user, machine, DC), that private key is the end-entity key (used for TLS, auth, decryption, signing as that identity).
    - If the certificate is a **CA cert** (Basic Constraints `CA:TRUE`, key usage `keyCertSign`/`cRLSign`), then the matching private key is the **CA signing key** — the key that can sign other certs.
---
### 2) How to tell whether the private key you found is a CA signing key or an end-entity key

Check these pieces of output on that cert:
- **Template / Certificate Type**
    - `Template: CA` or `Certificate Template Name (Certificate Type): CA` → CA cert.
- **Basic Constraints / Key Usage / EKU**
    - Look for `Basic Constraints: CA:TRUE` or `Key Usage` containing `keyCertSign` / `cRLSign`. That indicates the cert is allowed to sign other certificates.
- **Subject**
    - `Subject: CN=Certificate-LTD-CA...` strongly suggests it’s the CA cert, but subject alone is not proof — combine with Key Usage/Template.
- **Provider / Key Container**
    - Provider like `Microsoft Software Key Storage Provider` + `Private key is present` → private key stored in software KSP (potentially exportable).
    - HSM provider names indicate stronger protections.
- **Exportability**    
    - If `certutil -store` or `certutil -v -store` indicates the private key is exportable (or you can successfully `certutil -exportPFX`), you can extract the private key immediately.
---
### 3) Concrete `certutil` signs to look for (example lines)

- `Private key is present` → matching private key exists on that host.    
- `Private key is exportable` or `certutil -exportPFX` succeeds → you can extract it.
- `Template: CA` or `Certificate Template Name (Certificate Type): CA` → this cert is a CA cert.
- `Key Usage: keyCertSign, cRLSign` or `Basic Constraints: CA:TRUE` → CA signing capability.

If you see _both_ `Private key is present` **and** `Template: CA` / `keyCertSign`, then that host has the **CA private key**, not merely an end-entity key.

---
### 4) Quick commands you can run to programmatically check

PowerShell one-liners:
List certs and show whether private key exists and EKU/basic constraints:

```powershell
Get-ChildItem Cert:\LocalMachine\My | ForEach-Object { $thumb = $_.Thumbprint; $subj  = $_.Subject; $hasPK = if ($_.HasPrivateKey) {'yes'} else {'no'}; $ku = ($_.Extensions | Where-Object { $_.Oid.Value -in @('2.5.29.15','2.5.29.19') } ).Format($false) ;[pscustomobject]@{Thumb=$thumb; Subject=$subj; HasPrivateKey=$hasPK; KeyUsage=$ku} } | Format-Table -AutoSize

Get-ChildItem Cert:\LocalMachine\My | ForEach-Object {
  $thumb = $_.Thumbprint
  $subj  = $_.Subject
  $hasPK = if ($_.HasPrivateKey) {'yes'} else {'no'}
  $ku = ($_.Extensions | Where-Object { $_.Oid.Value -in @('2.5.29.15','2.5.29.19') } ).Format($false)
  [pscustomobject]@{Thumb=$thumb; Subject=$subj; HasPrivateKey=$hasPK; KeyUsage=$ku}
} | Format-Table -AutoSize
```

Check EKU / CA ability for a specific cert:

```powershell
# Try export (will prompt for password if allowed):
certutil -exportPFX My <certificate Serial number> Certificate-LTD-CA.pfx

# replace thumbprint
certutil -dump -v -store My <thumbprint>

# or
certutil -v -store My <cert-number-from-list>
```

---

### 5) Important caveats
- **Non-exportable ≠ safe**: a key flagged non-exportable can sometimes still be extracted by advanced tools (Mimikatz, memory/LSA extraction) especially for software KSP keys. HSM-backed keys are far safer.
- **Public certs are everywhere**: seeing a CA subject in the Trusted Root store is normal — that only shows the public certificate. You must find `Private key is present` for it to be dangerous.
- **Always confirm Key Usage**: Subject string alone is not enough to conclude it’s a signing key.
---
### TL;DR

- `certutil -store` telling you a _private key is present_ means the private key that **pairs with that certificate’s public key** exists on the machine.    
- If the certificate is a CA cert (Template/KeyUsage shows `CA`/`keyCertSign`), then the private key present is the **CA private key** (the key that can sign other certs).
- If it’s an end-entity cert (user/machine), then the private key present is that user/machine’s private key and **cannot** (by usage rules) be used to sign arbitrary certificates unless it is improperly configured as a CA.

If you want, I can parse your earlier `certutil -store` output and annotate exactly which lines mean what (point out the CA cert, the evidence the private key was present/exportable, and which of Ryan’s certs are end-entity only). Want me to do that?

You can use:
```powershell
Get-ChildItem Cert:\LocalMachine\My | ForEach-Object { $thumb = $_.Thumbprint; $subj  = $_.Subject; $hasPK = if ($_.HasPrivateKey) {'yes'} else {'no'}; $ku = ($_.Extensions | Where-Object { $_.Oid.Value -in @('2.5.29.15','2.5.29.19') } ).Format($false) ;[pscustomobject]@{Thumb=$thumb; Subject=$subj; HasPrivateKey=$hasPK; KeyUsage=$ku} } | Format-Table -AutoSize

```


---
