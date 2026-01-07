**ESC12** is a **misconfiguration** (and a design-level abuse scenario), not a single CVE that has a direct "patch."

While many early ESC techniques (like ESC1-ESC8) were popularized by the SpecterOps research in 2021, ESC12 was documented later (credited to Hans-Joachim Knobloch) as part of the continued discovery of how AD CS can be abused.

###  What is ESC12?
ESC12 focuses on **abusing shell access to CA servers to access a CA's private key even when it is stored in a Hardware Security Module (HSM).** In a standard "Golden Certificate" attack (DPERSIST1), if the CA's private key is stored in software, an attacker who gains local administrator access to the CA can simply export the key.1 However, many organizations use an **HSM** to prevent the private key from ever leaving the physical hardware. ESC12 demonstrates that if you are a local admin on the CA, you don't actually need to _steal_ the key—you can **legitimately use the HSM's provider** (KSP/CSP) to sign forged certificates while the key remains safely inside the hardware.

### Misconfiguration vs. CVE

- **It is a Misconfiguration/Design Issue:** It relies on the fact that a Local Administrator on a CA server inherently has the permissions required to interact with the Key Storage Provider (KSP) that talks to the HSM. There is no "bug" to fix; if you are an admin on the CA, you own the PKI.
    
- **It is not a CVE:** Unlike **Certifried (CVE-2022-26923)**, which was a specific logic bug in how SIDs were handled, ESC12 is a post-exploitation technique.2 If an attacker reaches the level of CA Admin, they have reached the "end of the game" for that PKI.
    

---

### How to "Mitigate" ESC12 (Since there is no patch)

Because ESC12 isn't a bug, you can't "patch" it. Instead, you must harden the environment to prevent the prerequisite (Local Admin on the CA):
1. **Tiered Administration:** Ensure your CA servers are treated as **Tier 0** assets. Only Domain Admins or a very small, highly audited group of PKI Admins should have local login rights.
2. **HSM Role Separation:** Some high-end HSMs allow "Administrator" vs. "Operator" roles. You can configure the HSM so that the Windows "System" account can _use_ the key to sign, but a separate physical smart card/quorum is needed to _manage_ the key.
3. **Monitor for Out-of-Band Certificates:** Use tools like **Microsoft Defender for Identity** or specialized PKI monitors to alert when the CA issues a certificate that doesn't correspond to a legitimate, logged enrollment request

### Summary Table

|**Feature**|**ESC12 Details**|
|---|---|
|**Type**|**Misconfiguration / Post-Exploitation Technique**|
|**Primary Goal**|Forge certificates using a CA's private key stored in an **HSM**.|
|**Prerequisite**|**Local Administrator** access on the Certificate Authority server.|
|**Patch Status**|**No patch.** This is intended functionality of how Windows interacts with HSMs.|

Would you like the **PowerShell** commands used to identify if a CA is using an HSM-backed provider versus a software one?