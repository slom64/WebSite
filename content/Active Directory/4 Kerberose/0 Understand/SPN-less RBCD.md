# Nutshell
- In S4u2self, the KDC need to know the requester SPN for 2 resones:
	1. To encrypt the forwardable TGS using the account password that hold the SPN
	2. To make sure the SPN have the right to do delegation.
	Thats why it fails to encrypt using user account by default without using SPN.
- In S4u2proxy, the KDC make assumption that the received forwardable ticket is from a service and need to be decrypted using the account password that holds the SPN.
- So, we use u2u to instructe the KDC to encrypt using the TGT session key, then we change the user password so that the KDC can decrypt the TGS forwardable.

---

In a normal Resource-Based Constrained Delegation attack, the service `WEBSRV` will initiate a `TGS-REQ` with the `S4U2Self` Kerberos extesnsion, asking the KDC to Impersonate user `DC01` to access itself.
If protocol transition is enabled, the `WEBSRV` service will get a `forwardable service ticket` for the user `DC01` to be used by the itself to access itself.
The `WEBSRV` then will initiate another `TGS-REQ` to Impersonate that user to access `DBSRV`, but this time with the `S4U2Proxy`, embedding a copy of the previously fetched `service ticket` for user `DC01`.
The KDC, will try to decrypt the `service ticket` sent by `WEBSRV` using the service's secret key, if succeeded, the KDC can then extract the embedded `service ticket` of `DC01`, and check if the service `WEBSRV` is allowed to Impersonate users to access `DBSRV`, if it is, then the KDC will send back a `service ticket` with `DC01` user's info, encrypted with `DBSRV` secret key, for `WEBSRV` to use to Impersonate that user and access `DBSRV`.

Now, to SPN-less RBCD Attacks.
If user `ahmed` wants to Impersonate `DC01` to access `DBSRV`, he will try to follow the same flow, `ahmed` will initiate a `TGS-REQ` with the `S4U2Self` Kerberos extension, asking the KDC to Impersonate user `DC01` to access itself.
The KDC will have no Idea which key to use to encrypt the `service ticket`, so it will fail in this case.
`ahmed`, to make this attack succeed, will use `S4U2Self` + `U2U` Kerberos Extensions.
`U2U` will tell the KDC:

> Encrypt the service ticket using ahmed's Ticket Session Key from his TGT

The KDC will encrypt the service ticket using ahmed's Ticket Session Key from his TGT, and sent it back to ahmed.
Now, when ahmed wants to go on with the attack, he will try to send a TGS-REQ with S4U2Proxy Kerberos Extension, embedding a copy of DC01 previously acquired service ticket in the request.
When the KDC receives the TGS-REQ, it will try to decrypt it using ahmed's secret key, as it has no Idea this was issued using the U2U Kerberos Extension, and the KDC will fail here.
To make this work, ahmed changes his password hash to the Ticket Session Key, in this case, when ahmed try again to request a TGS-REQ with S4U2Proxy Kerberos extension, the KDC will try to decrypt it with ahmed's secret key and will succeed, then the KDC will check if ahmed has permissions to Impersonate DC01 to access DBSRV, and if yes, it will send back a service ticket with DC02 user info, which ahmed will then use to access DBSRV

Limitations for this attack is:
 1. RC4 must be allowed for Kerberos,
 2. the attack will render the account used for it inaccessible,
 3. domain policy might prevent password reset without the intervention of a privileged account die to minimum password age requirements

---

> [!Question] 
> what i don't understand? why the KDC fail to know which key should it use to encrypt the TGS in S4u2self "it doesn't use the long term key" BUT it knows which key should it use when decrypt it for S4u2proxy "it use the long term key". that is wired in the s4u2self the kdc doesn't use the long term key but when trying to decrypt the TGS for s4u2proxy it use the long term key


The confusion here stems from the asymmetry in how Kerberos handles ticket issuance (encryption) versus validation (decryption) in the S4U extensions, especially when no SPN is involved. I'll break it down step by step to clarify why the KDC behaves differently in S4U2Self (encryption) compared to S4U2Proxy (decryption). This is based on the Kerberos protocol specs (e.g., MS-SFU) and how delegation works without a traditional service account.

### 1. **Quick Kerberos Basics on Ticket Encryption/Decryption**
   - **Service Ticket Issuance (TGS-REQ Response)**: When the KDC issues a service ticket (TGS), it encrypts the ticket with the *target principal's long-term key*. For a real service (e.g., WEBSRV with an SPN like HTTP/websrv.domain.com), this is the service account's NT hash-derived key. The requester (e.g., a user or another service) can't decrypt it—only the target service can, to prove authenticity.
   - **Ticket Validation/Decryption**: When a ticket is presented (e.g., in AP-REQ or embedded in S4U2Proxy), the recipient (service or KDC) decrypts it using its own long-term key to extract the contents, like the user's identity and authorizations.
   - Key point: The KDC always assumes tickets follow this model unless extensions like U2U override it.

### 2. **S4U2Self Without U2U (Why Encryption Fails Without SPN)**
   - In S4U2Self, the requester (ahmed) is asking the KDC: "Give me a forwardable service ticket impersonating DC01 to *myself* (ahmed) as the target service."
   - Normally, for a service with an SPN:
     - The KDC knows the target (WEBSRV) has a registered SPN and thus a long-term key.
     - It encrypts the TGS with WEBSRV's long-term key.
   - But for ahmed (no SPN):
     - Ahmed is a user account, not a registered service. There's no SPN mapping to a specific long-term service key for "impersonation to self."
     - The KDC looks for a target principal key to encrypt the ticket with—but since ahmed isn't treated as a service, it has no "service key" to use. It just has ahmed's user long-term key (NT hash), which isn't registered for service ticket encryption in this context.
     - Result: The KDC fails the request because it can't determine or use an appropriate encryption key for the ticket. It doesn't default to the user's long-term key here because S4U2Self is designed for *services* (with SPNs), not users. The protocol expects a service context for encryption.

   This is why plain S4U2Self fails without an SPN—it's not "wired" (weird); it's by design to prevent arbitrary users from impersonating others without proper setup.

### 3. **How U2U Fixes S4U2Self Encryption**
   - U2U (User-to-User) is an extension that changes the encryption behavior for scenarios where the target "service" might not have a traditional long-term key (or when peers are users).
   - When ahmed adds U2U to the S4U2Self request:
     - It tells the KDC: "Encrypt the impersonated TGS (for DC01 to ahmed) using the session key from *my current TGT* instead of a long-term key."
     - The KDC complies because U2U overrides the default. Now the ticket is encrypted with a short-term session key (derived from ahmed's TGT), not a long-term key.
     - Ahmed can decrypt it himself (since he has the TGT session key), but for delegation, he doesn't need to—he just embeds it as-is in the next request.
   - This bypasses the SPN/long-term key requirement for issuance, treating ahmed like a "peer" rather than a strict service.

### 4. **S4U2Proxy: Why Decryption Uses (or Tries) the Long-Term Key**
   - Now, ahmed sends the S4U2Proxy request: "Give me a ticket impersonating DC01 to DBSRV, and here's the embedded TGS from S4U2Self as evidence."
   - The KDC's job in S4U2Proxy is to *validate* the embedded TGS as legitimate "evidence" that DC01 "accessed" ahmed.
   - Validation process:
     - The embedded TGS claims to be a service ticket for the "front-end service" (ahmed).
     - **The KDC assumes this is a *standard* service ticket, which should be encrypted with the target principal's (ahmed's) long-term key—just like any other service ticket.**
     - So, it looks up ahmed's account and uses ahmed's long-term key (NT hash) to attempt decryption. It doesn't "know" this ticket was issued via U2U (the extension isn't carried over in a way that changes decryption assumptions).
     - If decryption succeeds, the KDC extracts DC01's info, checks delegation rights (msDS-AllowedToActOnBehalfOfOtherIdentity), and issues the new ticket to DBSRV.
   - But initially, it fails because the embedded TGS was encrypted with the TGT session key (from U2U), not ahmed's original long-term key. Mismatch!

### 5. **Why the Asymmetry Feels "Wired" (Weird) and How the Attack Resolves It**
   - **The Core Reason for the Difference**:
     - **Issuance (S4U2Self)**: Focused on *creating* a ticket for a target. Without SPN, no clear long-term key, so it fails unless U2U provides an alternative (session key).
     - **Validation (S4U2Proxy)**: Focused on *verifying* an existing ticket. The KDC treats the embedded ticket as if it were for a standard service principal, so it always tries the long-term key for decryption—regardless of how it was issued. It doesn't retroactively apply U2U logic; it assumes standard Kerberos semantics.
   - This is a protocol quirk: Kerberos prioritizes security by defaulting to long-term keys for services, but U2U is a bolt-on for flexibility. The attack exploits this gap.
   - **Resolution via Hash Change**:
     - Extract the session key (which is NTLM-compatible due to RC4).
     - Overwrite ahmed's long-term key (NT hash) to match that session key.
     - Now, when the KDC tries decryption in S4U2Proxy, it uses the "new" long-term key—which matches the encryption key from U2U. Success!

In summary, the KDC isn't being inconsistent—it's following different rules for ticket creation (flexible with extensions like U2U) versus verification (strict, assuming long-term keys). This design prevents abuse in normal flows but creates an exploitable edge case for SPN-less RBCD. If you share a specific tool command or log snippet where this happens, I can map it to these steps!