# ESC4 — Abuse of Template Permissions

### 1. What certificate templates are
- Templates = **blueprints** in AD CS that define _how a certificate should be issued_:
    - What EKUs (purposes) it has (e.g., logon, server auth).
    - Whether requesters can supply SANs.
    - Whether manager approval is required.
    - Whether authorized signatures are required.
    - Who is allowed to **enroll**.
- Templates are AD objects → they have a **security descriptor (DACL)** just like a user or group object.
---
### 2. What happens if you control a template
- If an attacker has **WriteDacl / FullControl** or even certain **WriteProperty** rights over a template, they can:
    - Change flags to allow SAN specification (ENROLLEE_SUPPLIES_SUBJECT).
    - Remove manager approval or signature requirements.
    - Add enrollment rights for low-privileged users.
    - Add dangerous EKUs (like Client Authentication or Smart Card Logon).
Essentially → you turn a _safe template_ into an **ESC1-like vulnerable template**.

---
### 3. ESC4 Abuse Requirements
To make privilege escalation possible, attacker modifies a template so it:
- ✅ Grants **Enrollment rights** to attacker or Everyone.
- ✅ Disables **Manager Approval** (`PEND_ALL_REQUESTS`).
- ✅ Removes **Authorized Signature** requirement (`mspki-ra-signature=0`).
- ✅ Enables **ENROLLEE_SUPPLIES_SUBJECT** (so requester can pick their SAN → impersonation).
- ✅ Adds EKU for **Authentication** (Client Authentication / SmartCard Logon / PKINIT).

Result: attacker can request a cert with SAN=Domain Admin → logon as DA.

---
### 4. Example Flow (ESC4 Abuse)
1. Attacker has **Write access to a certificate template** (e.g., “User Template”).
2. Attacker modifies template → allows SAN supply + client auth EKU.
3. Attacker enrolls in the template, requests cert with `SAN=administrator@corp.local`.
4. CA issues cert (no manager approval, no signature checks).
5. Attacker uses cert for PKINIT → gets TGT as Administrator.
---
### 5. Key difference from ESC1, ESC6, ESC9, ESC10
- **ESC1/6/9/10** = _already vulnerable templates or CA settings_.
- **ESC4** = _attacker makes the template vulnerable_, because they control its permissions.
👉 Think of it like this:
- ESC1 = “Door is already unlocked.”
- ESC4 = “You’ve got the master key, so you can change the lock however you want, then open it.”
---
# 🔹 Summary of ESC4

- **Cause**: Attacker has excessive rights over a template (DACL misconfig).
- **Abuse**: Modify template settings → turn it into ESC1-style vuln.
- **Impact**: Request certificates for privileged accounts → domain compromise.
- **Difference**: ESC4 isn’t about CA or template already being broken — it’s about _you having the power to break them_.
---

⚡️ ESC4 is especially dangerous in **delegated environments** where helpdesk or service admins have write rights to templates. That’s enough for escalation to DA if misused.

---

Do you want me to also compare ESC4 with **ESC7** (since ESC7 is about DACLs too, but on the CA object instead of templates)?