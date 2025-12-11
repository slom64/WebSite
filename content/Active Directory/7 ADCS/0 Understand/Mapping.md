Mapping could be:
- explicit
- implicit
	- Kerberos mapping `StrongCertificateBindingEnforcement`
		- `0`: no strong certificate mapping is realised. The `szOID_NTDS_CA_SECURITY_EXT` extension is not check and the authentication behavior is similar to what was done before the patch. This is the "Disabled mode". This option can't be enabled anymore.
		- `1`: default value after the patch. The KDC checks if the explicit certificate mapping is present (strong mapping). If yes, the authentication is allowed; if no, it checks if the certificate security extension is present and validate it. If it is not present, the authentication can be allowed if the user account predates the certificate. This is the "Compatibility mode"
		- `2`: the KDC checks if the explicit certificate mapping is present (strong mapping). If yes, the authentication is allowed; if no, it checks if the certificate security extension is present and validate it. If it is not present, the authentication is refused. This is the "Full Enforcement mode"
	- Schannel mapping`CertificateMappingMethods`
		- `0x0001`: Subject/issuer explicit mapping
		- `0x0002`: Issuer explicit mapping
		- `0x0004`: SAN implicit mapping. "This one alone is danger"
		- `0x0008`: S4USelf implicit Kerberos mapping
		- `0x0010`: S4USelf explicit Kerberos mapping
		- The current default value is `0x18` (`0x8` and `0x10`). Schannel doesn't directly support the `szOID_NTDS_CA_SECURITY_EXT` security extension, but it can utilize it by "converting" the Schannel certificate mapping to a Kerberos certificate mapping using S4USelf. The mapping process will then be performed as explained in the Kerberos mapping section.

---
# 🧩 What is “Certificate Mapping” in AD CS?

In Active Directory, a **certificate** is basically just a key + identity. But when you present a certificate to authenticate, the DC (Domain Controller / KDC) needs to answer a critical question:
👉 “**Which AD account does this certificate belong to?**”

That process = **certificate mapping**.

If mapping is weak/misconfigured, an attacker could use _their own_ certificate but trick the DC into mapping it to an _admin account_. That’s what ESC6, ESC9, ESC10, and the “Certifried” (CVE-2022-26923) bug are all about.

---

## 🔑 Two ways mapping works

### 1. **Explicit mapping**

- An admin explicitly links a cert to an AD object.
- This is stored in the AD attribute `altSecurityIdentities`.
- Example: user `Alice` has `altSecurityIdentities= X509:<I>CN=CorpCA…<S>CN=Alice`.
- When Alice presents that cert, the DC knows _this cert == Alice_.

👉 This is secure, but requires manual setup.

---

### 2. **Implicit mapping**

- The DC tries to guess which account the cert belongs to, based on certificate fields (Subject or SAN).
- Example:
    - Cert contains UPN = `alice@corp.local` → DC matches it to AD account `alice`.
    - Cert contains DNS = `pc01.corp.local` → DC matches it to computer account `PC01$`.

👉 This is much more dangerous, because if the rules are loose, an attacker can craft a cert that maps to someone else.

---

## 🧨 What went wrong historically (Certifried bug & ESC6/ESC9/ESC10)

Before Microsoft patched things, the DCs did **lazy implicit mapping**:

- If cert had a UPN field → map directly to that userPrincipalName.    
- If no match, try sAMAccountName.
- If no match, try with `$` → might map to a machine account.

🚩 Example attack:  
I make myself a cert with UPN = `administrator@corp.local`.  
DC happily maps it to the Domain Admin account.  
Now I can log in as DA using _my_ cert.

This was **CVE-2022-26923 (Certifried)**.

---

## 🔒 Microsoft’s fix: `szOID_NTDS_CA_SECURITY_EXT`

To close this hole, Microsoft added a new extension to certs issued by AD CS:
- **`szOID_NTDS_CA_SECURITY_EXT`** → contains the AD object’s **SID** at the time of enrollment.    
- So now when you present a cert, the DC checks:
    - Does this SID match the actual account?
    - If yes → allow.
    - If no → reject.

This prevents the “fake UPN = admin” trick, because your SID is baked in at issuance time.

---

## ⚙️ The Registry Keys

### 1. **StrongCertificateBindingEnforcement** (Kerberos mapping)

- Controls how strict Kerberos is when mapping certs. 
- Values:
    - **0 = Disabled** → old weak behavior (Certifried).
    - **1 = Compatibility mode** (default since patch):
        - If cert has explicit mapping → good.
        - If cert has the new SID extension → good.
        - If no extension → fallback to old weak behavior, _but only for old accounts_.
    - **2 = Full Enforcement** → only explicit mapping or SID extension allowed.

👉 In practice: 1 = safe but backwards compatible, 2 = strict, 0 = broken.

---

### 2. **CertificateMappingMethods** (Schannel mapping)

- This controls how mapping works for Schannel (TLS/SSL) auth.    
- It’s a **bitmask** (each bit is a method):
    - `0x0001` → Subject/Issuer explicit mapping
    - `0x0002` → Issuer explicit mapping
    - `0x0004` → SAN implicit mapping
    - `0x0008` → S4Uself implicit Kerberos mapping
    - `0x0010` → S4Uself explicit Kerberos mapping

Default = `0x18` (0x8 + 0x10 → only Kerberos-based mappings).  
Microsoft recommends = `0x1f` (all methods).
### 🧭 TL;DR

- **Certificate mapping = linking a cert to an AD account.**
- Used in Kerberos & Schannel for logon/auth.
- Weak implicit mapping = attackers can impersonate others by crafting malicious certs.
- Fix = use SID-based binding (`szOID_NTDS_CA_SECURITY_EXT`) + enforce via registry keys.
---


> [!question] 
> i have asked you some question about this before how to Validate Requester of certificate and you said that when we try to get certificate we should already have tgt ticket "kerbersoe authenticated". is that the same thing here? and sorry what is SAM, SID, UPN


## 🔑 "Requester validation" vs "Certificate mapping"

When you request a certificate from AD CS (via the CA), two different trust checks happen:

### **A. Certificate Enrollment (request stage)**

- You go to the CA and say: _"I want a cert!"_
- The CA has to check _who you are_ before issuing it.
- That’s where **Kerberos authentication** comes in:
    - You must already be logged in and have a **TGT (Ticket Granting Ticket)**.
    - The CA uses Kerberos to validate your identity (`you = Alice`).
- If allowed, the CA issues you a cert — and adds your **SID** into the new extension (`szOID_NTDS_CA_SECURITY_EXT`).  
    👉 This prevents someone just showing up unauthenticated and asking for certs.

---

### **B. Certificate Authentication (login stage)**

- Later, you use that certificate to log in to DC (Kerberos or Schannel).
- Now the DC must answer:  
    👉 _“This cert belongs to which AD account?”_
- That’s the **certificate mapping** step (explicit vs implicit).
So:
- Enrollment (CA side) = “are you really Alice?” ✅ requires Kerberos/TGT.
- Authentication (DC side) = “this cert corresponds to which account?” ✅ mapping step.

These are related but **different layers**.

---

## 🧾 2. The AD identity fields (SAM, SID, UPN)

When we talk about “mapping a cert to an account,” it means matching the cert’s info to one of these AD identifiers:

- **SAM (sAMAccountName)**
    - Legacy NT4-style username.
    - Example: `ALICE`, `BOB`, or `PC01$` (for a machine).
    - Often used in the form `DOMAIN\ALICE`.
    - Unique only inside a domain.
- **SID (Security Identifier)**
    - Unique ID assigned to every object in AD.
    - Looks like: `S-1-5-21-3623811015-3361044348-30300820-1013`.
    - Never reused.
    - Even if you rename Alice → Bob, her SID stays the same.
- **UPN (UserPrincipalName)**
    - Modern “logon name” that looks like an email.
    - Example: `alice@corp.local`.
    - Must be unique in the forest.

---

## 🔍 How this ties together

1. **Enrollment stage**
    - Alice requests a cert.
    - CA uses Kerberos → sees Alice’s SID.
    - Issues cert with `szOID_NTDS_CA_SECURITY_EXT = Alice’s SID`.
2. **Authentication stage**
    - Alice uses cert to log in.
    - DC sees cert → checks mapping:
        - If explicit mapping exists → match Alice.
        - If implicit mapping (UPN = alice@corp.local

---

> [!question] 
> i have multiple of things in my mind. why didn't microsoft used an ID for each certificate they issue and send this ID back to user and when he send it back we can use it to validate him. And another question we can't just modify the UPN in certificate to be administartor, because we modify in the cert which will corrupt its signature isn't?

## 🔹 1. Why not just give each certificate a unique ID?
Actually, X.509 _does_ have a **unique serial number** for every certificate issued by a CA.

- When the CA issues a cert, it sets:
    - **Serial Number** (unique per CA)
    - **Issuer Name** (the CA identity)
- The pair `(Issuer, Serial Number)` uniquely identifies a cert.

But… here’s the problem:

- If AD only relied on _certificate ID_, the DC would have to constantly call back to the CA every time a user logs in (to ask _“does serial 1234 belong to Alice?”_).
- That would make Kerberos/Schannel authentication **dependent on online CA checks** — slow and brittle.

Instead:

- They embed the **identity mapping information (UPN/SID)** _inside the certificate itself_.
- This way the DC can **validate offline** just by looking at the cert (and validating CA signature).

👉 That’s why AD uses mappings instead of a central lookup by serial number.  
It’s all about **performance and offline validation**.

---

## 🔹 2. “Why can’t we just change the UPN to Administrator?”

You’re right on target here 🚀

A certificate is basically:
```mathematica
{
   Subject = alice@corp.local,
   PublicKey = (Alice’s key),
   Issuer = Corp-CA,
   Signature = Sign(IssuerPrivateKey, Hash(Subject+PublicKey+...))
}

```
- If you **edit anything** (like changing `alice@corp.local` → `administrator@corp.local`),
- The **hash changes**,
- Which means the CA’s **digital signature no longer validates**,
- Which means DC will reject it.

So no, you cannot _directly_ tamper with a valid cert and keep it trusted.

---
## 🔹 3. Where does the attack come in then?

The trick isn’t to _edit_ an existing cert, but to **get the CA to issue you a new valid cert** with the malicious UPN.

Example (ESC6 / Certifried abuse):
- If the CA template allows **low-privileged users** to request certs without proper identity validation…
- You request a cert where the `SubjectAltName.UPN = administrator@corp.local`.
- The CA signs it happily (because the template rules are weak).
- Now the cert is 100% valid and signed — DC trusts it.
- You authenticate as Domain Admin. 🚨

So the vulnerability isn’t forgery (editing certs), it’s **mis-issuance** by a trusted CA.

---

✅ Summary:

- Certificates already have IDs (serials), but using them for online lookup would kill performance → so identity info (UPN/SID) is embedded in the cert.
- You can’t just edit a cert’s UPN (signature breaks).
- Attacks work by _convincing the CA to issue a legit cert with attacker-controlled identity_.

---

> [!question] 
> hmmm, then why is UPN is user controllable variable, why not after authenticate the user, the CA just append his UPN (or any thing we can use to identify him). doing so, we can make sure that the CA who issued the certificate and nothing is tampered

💯 Excellent thinking — you’re basically pointing out the _design gap_ that Microsoft had to fix after **Certifried (CVE-2022-26923)**. Let’s break it down.

---

## 🔹 Why was UPN “user-controlled”?

Originally (before the patches), AD CS worked like this:
1. **Client generates CSR (Certificate Signing Request)**
    - Client picks subject info (like `CN=Bob` or `UPN=bob@corp.local`).
    - Client includes their _public key_.
    - Client signs CSR with their _private key_ (proving ownership of the key).
2. **Client submits CSR to CA**
    - CSR says: “Please issue me a cert with this UPN.”
3. **CA checks authorization rules from the certificate template**
    - Example: "Users in group X can enroll in this template."
    - BUT ❌ the CA did **not validate whether the UPN inside the CSR actually matched the authenticated user**.
Result:
- Any authenticated user could request a cert with **UPN=administrator@corp.local**
- If the template allowed domain auth EKU → boom, you could log in as Domain Admin.
- This is what **ESC6 / Certifried** abuses.
---
## 🔹 Why not just have CA fill in the UPN automatically?

That’s _exactly_ the mitigation Microsoft introduced.
Now, after the patch:
- The CA **ignores UPN values in the CSR provided by the user**.
- Instead, the CA queries **Active Directory** for the authenticated requester’s attributes (like `userPrincipalName`, `sAMAccountName`, and `objectSid`).
- It then stamps those values into the issued certificate itself.

This ensures:  
✅ The identity bound to the cert is tied to the authenticated account.  
✅ The requester cannot lie about being another user.

---

## 🔹 So why did MS allow user-supplied UPNs originally?

Legacy reasons:
- X.509 PKI standards allow flexibility (subjects can request many identifiers).
- In older systems, CAs often relied on _trusting the CSR contents_.
- AD CS tried to follow that standard, but in an AD-integrated environment, it was too trusting.

The patch (adding `szOID_NTDS_CA_SECURITY_EXT` with the `objectSid`) basically made AD CS behave the way you suggest:  
👉 “Ignore what the user asks for, bind the cert identity to what AD says they are.”

---

✅ Summary:

- Before patch: UPN in CSR was user-controlled → led to ESC6/Certifried.
- After patch: CA injects UPN/SID itself based on the authenticated requester.
- This closes the gap: cert identity now comes from AD, not the attacker’s CSR.
---

> [!question] 
> ok cool you have gave me solid background, now i want to go back. what do we mean by kerberos mapping and schannel mapping. is that kerberos mapping we use StrongCertificateBindingEnforcement registry key to check how much aggressive we should be for dealing with UPN, but my question is when does kerberose mapping happens, or is that a mode we use and what is schannel is that the https?


Perfect question — you’re tying it all back together now 👌. Let’s slow it down and separate the concepts:

---

## 🔹 1. What is **Kerberos mapping**?

This happens when you try to authenticate to AD **using a certificate** instead of username+password.
- Example: Alice has a smartcard or a cert from AD CS.
- She tries to log in to a Windows machine or domain service.
- The **KDC (Kerberos Domain Controller)** must figure out:  
    _“Which AD account does this certificate belong to?”_

That mapping step = **Kerberos certificate mapping**.

👉 This is where the `StrongCertificateBindingEnforcement` registry key comes in.  
It tells the KDC **how strictly it should validate the certificate’s identity information**:
- **0 (Disabled mode)** → Trust UPN/DNS inside cert, even if not strongly bound. (Legacy, insecure → ESC6 worked here).
- **1 (Compatibility mode)** → Prefer explicit mapping (altSecurityIdentities) or the `szOID_NTDS_CA_SECURITY_EXT` extension (with SID). If missing, still fallback to weak UPN/DNS mapping.
- **2 (Full enforcement)** → Only accept strong mapping (SID in extension or explicit altSecurityIdentities). No fallback.

So:  
📍 **Kerberos mapping happens during certificate-based logon.**  
It decides which AD account the KDC associates with the presented certificate.

---

## 🔹 2. What is **Schannel mapping**?

Yes — Schannel = Microsoft’s **Secure Channel SSP** (Security Support Provider).  
It is used for:
- **TLS/SSL in Windows** (IIS, LDAP over SSL, RDP with TLS, SMB with signing, etc.).
- It’s the Windows component that does HTTPS-like cert validation and authentication.

When a client presents a certificate over TLS/SSL to a Windows service (e.g., LDAPS, IIS with client cert auth), Schannel must also decide:  
_“Which AD account maps to this certificate?”_

That’s **Schannel mapping**.

👉 Controlled by the `CertificateMappingMethods` registry key.

- Different flags = how Schannel tries to match certs to users:
    - `0x0001`: Subject/issuer explicit mapping (altSecurityIdentities).
    - `0x0004`: SAN implicit mapping (UPN/DNS from the cert).
    - `0x0008`: Kerberos S4U self implicit mapping (use Kerberos translation).
    - `0x0010`: Kerberos S4U self explicit mapping.
- Default = `0x18` (0x8 + 0x10 → mostly Kerberos S4U-based mappings).

If Schannel can’t directly map, it can hand off to Kerberos mapping logic (through S4U).

---

## 🔹 3. When do these happen in practice?

- **Kerberos mapping** → Smartcard logon, cert-based workstation/domain logon. (e.g., “Log on to Windows with a smartcard”).
- **Schannel mapping** → Certificate-based TLS authentication to Windows services. (e.g., Client cert auth to IIS/LDAP, or RDP with smartcard).

Both solve the same problem: _“What AD account does this cert belong to?”_  
But they happen in different security providers:

- Kerberos SSP → logon & tickets.
- Schannel SSP → TLS/SSL & client certs.

---

✅ **Summary**

- **Kerberos mapping** = KDC maps cert → user account (logon). Controlled by `StrongCertificateBindingEnforcement`.    
- **Schannel mapping** = Windows TLS/SSL maps cert → user account (client cert auth). Controlled by `CertificateMappingMethods`.
- Schannel can rely on Kerberos mapping as a backend if needed (via S4U).

---

> [!question] 
> yes lets do step-by-step flow of certificate-based logon with Kerberos vs TLS with Schannel. specially in TLS. i mean do we send the certificate in encrypted channel or we just send the certificate so it can know us. give me explain so i can have general understand of this


Nice — let’s walk both flows step-by-step and point out **exactly** where certificates are sent, who verifies what, and where _mapping_ (Kerberos vs Schannel) happens. I’ll keep it practical and call out TLS 1.2 vs TLS 1.3 differences since that’s a common source of confusion.

---

# 1) Certificate-based **Kerberos** logon (PKINIT) — high level flow

Used when a client obtains a Kerberos TGT using a certificate (smartcard / client cert).
1. **Client has:** a certificate (issued by enterprise CA) + corresponding private key. The cert may contain UPN/SID or the new `szOID_NTDS_CA_SECURITY_EXT` SID extension.
2. **Client prepares AS-REQ with PKINIT data:** the request includes the client certificate (or certificate chain) plus a signed AuthPack (a structure signed with the client’s private key) proving possession of the private key.
3. **KDC receives request & validates:**
    - Verifies the certificate chain (server finds intermediates or uses AIA) up to a trusted enterprise CA.
    - Checks certificate properties: EKU (PKINIT/Client Authentication), validity, revocation (CRL/OCSP).
    - Verifies the AuthPack signature using the public key from the presented cert → proves key possession.
4. **KDC performs certificate → account mapping:**
    - Uses explicit mapping (altSecurityIdentities) if present.
    - Or, depending on `StrongCertificateBindingEnforcement`, checks the certificate’s SID extension (strong binding) or falls back to implicit mapping using UPN/sAMAccountName (compatibility/legacy behavior).
5. **If mapping & authorization OK → KDC issues a TGT (AS-REP)** and the client can request service tickets as usual.

**Where the mapping happens:** inside the KDC when it receives the PKINIT AS-REQ and the client certificate. `StrongCertificateBindingEnforcement` controls how strict that mapping is.

---
# 2) Certificate-based **TLS** auth (Schannel) — step-by-step (server asking for client cert)

Used when a TLS server requests a client certificate (IIS, LDAPS, mutual TLS, etc.).
### TLS 1.2 (common in many enterprises)

1. **Client → Server:** `ClientHello` (in clear).
2. **Server → Client:** `ServerHello`, `Certificate` (server cert), and optionally `CertificateRequest` (asking the client for a client cert). These handshake messages are sent **in clear** prior to ChangeCipherSpec in TLS 1.2.
    - Because these are in clear, the server certificate and any client certificate the client will later send can be observed by an on-path eavesdropper.
3. **Client, if requested, sends:** `Certificate` (client’s certificate chain) **in clear**, then `ClientKeyExchange`, `CertificateVerify` (signature created with client private key proving possession), then `ChangeCipherSpec`, `Finished`.
4. **Server validates client certificate:**
    - Verifies certificate chain up to a trusted CA (uses local trust store + provided intermediates).
    - Checks EKUs, key usages, validity, CRL/OCSP.
    - Verifies `CertificateVerify` signature to ensure client holds the private key.
5. **Schannel maps cert → account:** Schannel (or the service) uses certificate mapping configured on the server:
    - explicit mapping (altSecurityIdentities),
    - implicit SAN/UPN mapping,
    - or translates to Kerberos mapping via S4U mechanisms (depending on `CertificateMappingMethods` flags).
6. **If mapping succeeds and access allowed → TLS session established and the server treats the client as that mapped account.*
### TLS 1.3 (better privacy)
- In TLS 1.3 the handshake changes: handshake keys are derived earlier and the server & client `Certificate` messages are **encrypted** with handshake keys.
- **Effect:** Certificates and Client Certificate messages are **not sent in plaintext** (improves privacy).
- Validation and CertificateVerify still happen, and Schannel mapping occurs after successful certificate verification — same mapping logic as above

**Short answer to your question “is certificate sent in encrypted channel?”**
- TLS 1.2: **No** — the Certificate messages are sent in cleartext (but the client proves possession with a later signed `CertificateVerify`).
- TLS 1.3: **Yes** — certificate messages are encrypted after keys are derived.
---
# 3) What exactly is being verified (both flows)

- **Chain verification:** validate the issuer chain up to a CA in the trust store (verify signatures with CA public keys).    
- **Key usage / EKU:** ensure the cert is allowed for the intended purpose (e.g., ClientAuth, SmartCardLogon, PKINIT).
- **Validity:** dates, revocation (CRL/OCSP) checks.
- **Proof of possession:** client signs handshake (TLS `CertificateVerify`) or PKINIT AuthPack — proves they own the private key matching the public key in the cert.
- **Mapping:** once the cert is valid and possession proven, map the cert to an AD account (explicit altSecurityIdentities or implicit SAN/UPN/SID rules).
---
# 4) Where mapping differences matter: Kerberos vs Schannel

- **Kerberos mapping** (KDC): used during PKINIT logon; controlled by `StrongCertificateBindingEnforcement`. It decides whether to require the SID extension / explicit mapping or allow legacy UPN mapping.
- **Schannel mapping** (TLS): used for TLS client auth to services; controlled by `CertificateMappingMethods`. Schannel can do explicit mapping, SAN mapping, or hand off to Kerberos mapping (S4U) in some modes.
---
# 5) Practical implications / security notes

- Even if a certificate is sent in clear (TLS 1.2), an attacker cannot _forge_ it without the CA’s private key. The client must prove possession of the private key (signature), so simply replaying a cert is not enough.
- TLS 1.3 reduces privacy leakage by encrypting certificates in the handshake.
- **The weak spot historically** was _how the KDC or Schannel maps certificate fields to accounts_ — implicit mapping via UPN allowed attackers to get a CA to issue a cert with `UPN=administrator` (or in some pre-patch flows to map attacker cert to admin). That’s why Microsoft introduced the SID extension and the registry enforcement modes.    
---
# Quick cheat-sheet (one-line)

- **PKINIT (Kerberos):** client sends cert inside AS-REQ → KDC verifies + maps → issues TGT. Mapping strictness = `StrongCertificateBindingEnforcement`.
- **TLS (Schannel):** server requests cert during TLS handshake → cert & proof-of-possession exchanged → server/Schannel verifies + maps → session authenticated. Mapping behavior = `CertificateMappingMethods`.
- **TLS 1.2:** certs in clear; **TLS 1.3:** certs encrypted.
---


> [!Question] 
> so certificate mapping happens when like the user try to authenticate using his certificate, then the KDC checks the certificate is valid then it looks at the sid to do mapping to account based on it (when implicit mapping is used), and what make this better than the older implementation (before certifried) is that the server will append the right sid of the user and won't trust the upn of the user

Yes, that's a solid grasp of the process—I'll refine and expand on it based on how certificate mapping works in Active Directory Certificate Services (ADCS), especially post-Certifried (CVE-2022-26923). I'll break it down step by step, including why the newer implementation is more secure. Note that as of December 2025, we're fully in "Full Enforcement" mode for these mappings (per Microsoft's updates: Disabled mode was phased out in April 2023, Compatibility mode ended in September 2025, and everything now defaults to strict enforcement).

### When and How Certificate Mapping Happens
Certificate mapping occurs during the authentication phase, specifically when a user (or machine) presents a certificate to authenticate to the domain—e.g., via Kerberos (for things like obtaining a Ticket-Granting Ticket or TGT) or Schannel (for TLS-based services like LDAPS). The Key Distribution Center (KDC) on the domain controller handles Kerberos auth, while Schannel handles TLS/SSL scenarios but often delegates to Kerberos via S4U (Services for User).

Here's the high-level flow:
1. **Certificate Validation First**: The KDC (or Schannel) checks if the certificate is inherently valid. This includes:
   - Verifying the signature and chain back to a trusted root CA.
   - Checking revocation status (via CRL/OCSP).
   - Ensuring it's within its validity period and has appropriate EKUs (e.g., Client Authentication).
   If it fails here, auth is denied—no mapping even attempted.

2. **Mapping the Certificate to an AD Account**: Once validated, the system needs to link the cert to a specific AD object (user or computer). This is where mapping comes in, and it can be explicit or implicit:
   - **Explicit Mapping**: The cert's details (e.g., issuer + serial number) must exactly match an entry in the AD object's `altSecurityIdentities` attribute. This is admin-configured and doesn't rely on the cert's contents beyond basic identifiers.
   - **Implicit Mapping** (what you're focusing on): No `altSecurityIdentities` entry needed. Instead, the system uses info embedded in the cert (like SAN fields or the SID extension) to look up the matching AD account.

   In implicit mapping under modern enforcement:
   - The KDC prioritizes the SID from the `szOID_NTDS_CA_SECURITY_EXT` extension (OID: 1.3.6.1.4.1.311.25.2).
   - It extracts the SID and queries AD for a matching `objectSid` on a user/computer object.
   - If it matches, mapping succeeds, and auth proceeds (granting tickets based on that identity's privileges).
   - If no SID extension or it doesn't match, it falls back to explicit mapping (if present) or other strong methods—but in Full Enforcement mode (current as of 2025), weak implicit mappings (e.g., just UPN or DNS) are rejected outright.

This happens transparently during auth attempts—no separate "mapping request" step.

### Why the Post-Certifried Implementation Is Better
The older implementation (pre-May 2022 patch for CVE-2022-26923) relied heavily on weak implicit mappings, which were vulnerable. Here's the contrast:

- **Old Way (Pre-Certifried/Weak Mappings)**:
  - Implicit mapping trusted user-supplied or easily spoofable info in the cert's SAN, like UPN (e.g., user@domain.com) or DNS name (e.g., host.domain.com).
  - The KDC would parse the SAN, search AD for a matching `userPrincipalName` (for users) or derive a `sAMAccountName` (e.g., appending "$" for machines), and map accordingly.
  - Problem: If a certificate template allowed the enrollee to specify arbitrary SANs (e.g., via `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT_ALT_NAME`), an attacker could request a cert with a spoofed UPN/DNS mimicking a privileged account (like a domain admin). The KDC would trust it without verifying the requester's true identity, enabling impersonation (core of exploits like Certifried or ESC1).
  - No built-in way to enforce "who really requested this?"—it was all based on the cert's self-reported details.

- **New Way (Post-Certifried/Strong Mappings with SID Extension)**:
  - The CA (during issuance) automatically embeds the requester's actual `objectSid` from AD into the cert via the SID extension. This happens only for enterprise CAs integrated with AD—the CA queries AD for the enrollee's SID and adds it, so it's not user-supplied or forgeable.
  - During auth, the KDC extracts and validates this SID directly against AD, ensuring the cert maps only to the exact account that requested it.
  - Even if the SAN has a UPN/DNS, the SID takes precedence in enforcement modes. Weak mappings (relying solely on UPN/DNS) are ignored or fail in Full Enforcement.
  - Security wins:
    - **Anti-Spoofing**: SIDs are unique, non-reusable, and domain-assigned—not something an attacker can fake in a request. Unlike UPNs, which can be arbitrary strings in misconfigured templates.
    - **No Trust in User Input**: The CA enforces the SID based on the authenticated enrollee's identity (via NTLM/Kerberos during enrollment), so even if SAN is enrollee-supplied, the SID overrides it for mapping.
    - **Backdating Protection**: The system can check if the cert predates the account (flagging potential fakes), with no fallbacks in Full Enforcement.
    - **Broader Coverage**: Addresses CVEs like 2022-26923 (Certifried), 2022-26931, and 2022-34691 by mandating strong bindings, reducing risks in VPN, Wi-Fi, or smart card auth.

In short, the old system was like trusting a self-declared ID on a badge; the new one is like embedding a verifiable, tamper-proof biometric (the SID) that the issuer (CA) stamps based on AD records. This shift makes attacks much harder, as exploiting templates alone isn't enough—you can't bypass the SID check. If your environment still has legacy certs without SIDs, they'd fail auth now, so re-issuance with modern templates is key.
