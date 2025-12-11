## Understanding ESC2

When a certificate template specifies the Any Purpose Extended Key Usage (EKU) or does not identify any Extended Key Usage, the certificate can be used for any purpose (client authentication, server authentication, code signing, etc.). it can be used as a requirement to request another certificate on behalf of any user.

---

 most explanations assume you already know what **Enrollment Agents**, **Subordinate CAs**, **EKUs**, and **certificate chain trust** are.  
Let’s break this down _from zero → to final exploit flow_ in a way that _fully connects everything_.

---

# 🧩 First — Missing Concepts You Need to Understand

### 1. **What is an Enrollment Agent?**

An Enrollment Agent is a user or certificate that is allowed to request certificates _on behalf of other users_.

Normally:  
You → request certificate → you authenticate as yourself  
But an **Enrollment Agent** can do:

```
I request a certificate FOR alice (or even Domain Admin)
```

🔥 So if an attacker becomes an Enrollment Agent, they can generate a cert for _any account they want_.  
That means **you can impersonate high-privilege users.**

---

### 2. **What is a Subordinate CA?**

There are two types of CA:

|CA type|Meaning|
|---|---|
|**Root CA**|Top authority. Trusted automatically.|
|**Subordinate CA**|A CA _signed by another CA_. Sub-CA can issue certificates too.|

A certificate template **with no EKU** or **Any Purpose EKU** behaves like a Subordinate CA certificate:

It can sign certificates **for anything**.

---

### 3. **EKU (Extended Key Usage)**

EKU defines _what a certificate is allowed to do_.

Examples:

|Purpose|EKU|
|---|---|
|Client Authentication|`1.3.6.1.5.5.7.3.2`|
|SmartCard Logon|`1.3.6.1.4.1.311.20.2.2`|
|Code Signing|`1.3.6.1.5.5.7.3.3`|
|Any Purpose (ESC2 issue!)|`2.5.29.37.0`|
|Enrollment Agent|`1.3.6.1.4.1.311.20.2.1`|

If a certificate has a **specific EKU**, it's restricted.  
If it has **NO EKU** or **Any Purpose EKU → it can act as _all EKUs at once_**.

That means:  
✔ Client auth  
✔ Code signing  
✔ Server auth  
✔ **Act as Enrollment Agent** (this is the vulnerability)

---

# 🔥 Now ESC2 Made Simple

### ESC2 happens when:

|Condition|Why it's dangerous|
|---|---|
|Template has **no EKU** or **Any Purpose EKU**|Makes you a _potential enrollment agent_|
|Low-priv users can enroll in it|Anyone can get that powerful cert|
|There exists another template that allows Enrollment Agent issuance|Lets you request certs FOR other users|

Thus:

```
You → enroll → get AnyPurpose certificate
Your cert can now impersonate other users!
You request a cert for Domain Admin
You now authenticate AS Domain Admin
```

This is privilege escalation.

---

# 🧨 ESC2 in a Single Sentence

If a low-priv user can enroll in a certificate template with **Any Purpose / no EKU**, they can request certificates _as other users_ — including Domain Admin — and authenticate as them.

---

# Why HTB Says "But subordinate CA cert cannot authenticate"?

Because:

⚠ If the CA is not trusted in **NTAuthCertificates**  
then certificates it signs cannot be used for logon.

So ESC2 may give you power to sign certificates (sub-CA style),  
but not necessarily authenticate to domain controllers unless trust exists.

But even when auth is blocked —  
you can still abuse it for:

|Abuse|Why useful|
|---|---|
|Code signing|Sign malware to look legitimate|
|Server authentication|Spin up fake services|
|TLS MITM|Fake certificate for HTTPS spoofing|

---

# Summary in the Simplest Possible Words

|Term|Meaning|
|---|---|
|Subordinate CA cert|A cert that can sign other certs|
|Enrollment Agent|Someone who can request certs for other users|
|No EKU / Any Purpose|The cert can act as **ALL EKUs**, including Enrollment Agent|
|ESC2|Low-priv user becomes Enrollment Agent → impersonates Domain Admin|

---


> [!question] 
> so in order to abuse the ESC2 i should be trusted as subordinate ca or should i include my self in NTAuthCertificates so it can be used for authentication

Great question — and this is the point that most people get stuck on.  
Let’s separate ESC2 exploitation into **two different goals**, because each requires something different.

---

# 🔥 Goal 1 — Privilege Escalation to Another User (e.g. Domain Admin)

You do **NOT** need to add yourself as a trusted subordinate CA.  
You do **NOT** need NTAuthCertificates.

For privilege escalation, here's the key:

### You only need your **AnyPurpose / No-EKU certificate** to act as an Enrollment Agent.

Then you request a certificate **on behalf of another user** through a template that allows agent-based enrollment.

Example:

```
certipy req -template User -on-behalf-of Administrator -pfx anypurpose.pfx
```

This gives you **a new certificate for Administrator**, and **THIS** certificate _is trusted_ (because it comes from the real Enterprise CA).

Then you authenticate:

```
certipy auth -pfx administrator.pfx
```

⚠ Your attacker certificate doesn't need NTAuthCertificates.  
The **second certificate you generated does**, and it already will be.

✔ So ESC2 → PrivEsc → No NTAuth modification needed.

---

# 🔥 Goal 2 — Use Your Own Certificate to Authenticate Directly (From ESC2 Cert)

This is where NTAuthCertificates matters.

A certificate used for smartcard/PKINIT logon must chain back to a CA trusted in:

```
CN=NTAuthCertificates,CN=Public Key Services,...
```

If your ESC2 certificate is treated like a **Subordinate CA certificate**, then:

| If your CA cert is not in NTAuthCertificates | ❌ Can't logon |  
| If you manually add it | ✔ You can authenticate directly |

But normally you **don't need this** for ESC2 exploitation.

Because it's easier and cleaner to:

1. Use ESC2 cert → act as Enrollment Agent
2. Request cert for Domain Admin
3. Authenticate using THAT cert
---
#  Summary Table

| Action                                            | Requires AnyPurpose cert | Requires NTAuthCertificates trust? |
| ------------------------------------------------- | ------------------------ | ---------------------------------- |
| Act as Enrollment Agent                           | ✔ YES                    | ❌ No                               |
| Request cert for Domain Admin                     | ✔ YES                    | ❌ No                               |
| Authenticate _as Domain Admin_ (using final cert) | (indirectly yes)         | ✔ Already trusted                  |
| Authenticate directly using ESC2 cert             | ✔ YES                    | ⚠ Only if you add to NTAuth        |

So your question answered simply:

> **To exploit ESC2 for escalation → NO, you do not need NTAuthCertificates or subordinate CA trust.**  
> You only need those **if you want your attacker-issued certificates to logon directly**.
