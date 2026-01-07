# ESC7 — Vulnerable Certificate Authority Access Control

### 1. The CA itself has permissions

The Enterprise CA object in AD (and its service) has its own set of permissions. Two of the most important rights are:
- **ManageCA** → like being a **CA admin**. Lets you change CA-wide settings.
- **ManageCertificates** → like being a **CA officer / cert manager**. Lets you approve or deny pending cert requests.
---
### 2. Why these rights are dangerous
- **With ManageCA:**
    - You can call methods like `ICertAdminD2::SetConfigEntry`.
    - That lets you flip CA-level flags such as **EDITF_ATTRIBUTESUBJECTALTNAME2**.
    - If you enable that flag → suddenly _all templates_ accept SAN attributes in requests.
    - That turns basically _any cert request into ESC6_.
    - Result: You can request a cert with SAN=Administrator → escalate to DA.
- **With ManageCertificates:**
    - Some templates are configured with **Manager Approval required**.
    - Normally, that means your cert request sits in “Pending” until a CA officer approves it.
    - If you have ManageCertificates, you can approve your _own_ pending requests.
    - That bypasses one of the main safety checks → effectively nullifying manager approval.

---
### 3. ESC7 Abuse Requirements
- If attacker has **ManageCA** → they can enable SAN requests globally → abuse like ESC6.
- If attacker has **ManageCertificates** → they can approve their own or other attacker requests → bypass protections.

---
### 4. Example Flow (ManageCA abuse)
1. Attacker compromises an account with **ManageCA** rights.
2. Attacker flips `EDITF_ATTRIBUTESUBJECTALTNAME2=1`.
3. Now templates that didn’t allow SANs before will accept them.
4. Attacker requests cert with SAN=[administrator@corp.local](mailto:administrator@corp.local).
5. CA issues cert → attacker uses it to logon as DA.

---
### 5. Difference ESC4 vs ESC7
- **ESC4** = Attacker controls **templates** (DACLs on templates in AD).
- **ESC7** = Attacker controls **the CA object itself** (ManageCA / ManageCertificates rights)

👉 ESC4 is “I can edit the blueprints for certs.”  
👉 ESC7 is “I can reconfigure the entire factory or approve certs myself.”

---

### 6. Big Picture

ESC7 is less common than ESC4 in misconfigs, but when it happens it’s **game over**:
- CA admins can make any template vulnerable.
- Cert managers can approve any request.
- Both situations give attackers the ability to mint their own trusted certs → impersonation → domain compromise.
---
⚡ So:
- **ESC4 = Template-level misconfig abuse.**
- **ESC7 = CA-level misconfig abuse.**
---

> [!question] 
> if i have ManageCA over CA and managa Certificates does that mean i can make change in the configurations of the CA like editing registry to make the CA vulnerable to ESC6

Yes, you have the right idea.

If you have **`ManageCA`**, you can absolutely change the configurations that control the Certificate Authority's behavior.

While you generally cannot open `regedit` and modify the registry remotely (unless you are a Local Admin), **`ManageCA`** allows you to interact with the CA Service (via RPC/DCOM), and **the CA Service itself has the authority to write these changes to its own registry keys.**

Here is the breakdown of how this specific "ESC7 to ESC6" pivot works.
### 1. The "Registry" Edit

You are not editing the registry directly; you are telling the CA to do it.
- **The Goal:** You want to enable the `EDITF_ATTRIBUTESUBJECTALTNAME2` flag (which causes the ESC6 vulnerability).
- The Location: This setting is stored in the registry on the CA server, specifically inside the Policy Module configuration:
    HKLM\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA Name>\PolicyModules\ClmPolicy\EditFlags
- **Your Permission:** Because you have `ManageCA`, the CA service accepts your command to update this "EditFlags" value.    
### 2. How to do it (PowerShell / Native Commands)
Since you prefer PowerShell/native commands, here is how you would exploit this using the native `certutil` tool, which is already present on Windows. You do not need `Certipy` for this part if you are operating from a Windows machine.

**Step 1: Check the current flags**
```
# Query the remote CA for the current 'EditFlags' value
certutil -config "CA-COMPUTER-NAME\CA-NAME" -getreg policy\EditFlags
```

Step 2: Enable the Vulnerability (ESC6)
You can use certutil to append the vulnerable flag (EDITF_ATTRIBUTESUBJECTALTNAME2) to the existing configuration.
```
# +EDITF... adds the flag. (-EDITF... would remove it)
certutil -config "CA-COMPUTER-NAME\CA-NAME" -setreg policy\EditFlags +EDITF_ATTRIBUTESUBJECTALTNAME2
```

### 3. The Critical "Gotcha": The Service Restart
This is the most common point of failure for this specific attack path in HackTheBox or real engagements.
Even though you have successfully updated the registry key via the CA service, **Policy Module changes usually do not take effect until the CertSvc service is restarted.**
- **The Problem:** Having `ManageCA` grants you rights to configure the CA, but it does **not** always grant you the right to restart the service (which is a Windows Service Control Manager permission).
- **The Result:** You might enable the flag, but the CA will ignore it until the server reboots or an admin restarts the service.
- **The Workaround:**
    - If you are lucky, `ManageCA` might coincide with other rights that allow service restart.
    - Otherwise, attackers often use ESC7 for a different path: **making themselves an "Officer"** to approve a malicious certificate request for a "Subordinate CA" template (which is usually pending approval), rather than trying to force ESC6.

### Summary
- **Can you change the config?** Yes. `ManageCA` lets you update the `EditFlags` which creates the ESC6 vulnerability.    
- **Does it work instantly?** Usually **No**. It requires a service restart.
- **Is it effectively editing the registry?** Yes, via the CA API.

---

> [!Question]
> if i can enable ESC6 that mean i can also enable ESC16 if i have manageCA
The short answer is **Yes**. If you have **`ManageCA`** privileges, you can potentially enable the conditions required for **ESC16**.

While ESC6 involves enabling a dangerous setting (the SAN flag), **ESC16** (which typically refers to exploiting **Weak Certificate Mappings** in the context of the KB5014754 patch) involves **disabling** a security feature.

### 🛡️ What is ESC16?

ESC16 is a newer classification that deals with **Certificate Mapping** abuse, specifically bypassing the security updates Microsoft released in 2022 (KB5014754).
- **The Fix:** Microsoft introduced a new OID (Object Identifier) called the **SID Extension**.1 When a CA issues a certificate, it embeds the user's SID into the certificate.2 The Domain Controller checks this SID to ensure the certificate actually belongs to that user.
- **The Vulnerability:** If a certificate _lacks_ this SID, and the Domain Controller is not in "Strict Enforcement" mode, the DC might fall back to checking the `userPrincipalName` (UPN) or `dNSHostName`. This allows an attacker to spoof any user if they can get a certificate with the victim's UPN but _without_ the victim's SID.
### ⚔️ How `ManageCA` enables it

If you have **`ManageCA`**, you can configure the CA to **stop issuing the SID Extension**. This "downgrades" the security of the issued certificates, making them vulnerable to mapping attacks again (assuming the Domain Controllers are not yet in strict enforcement mode).

Here is the attack flow using native commands (as you prefer):

#### 1. Check if the "SID Extension" is currently enabled (Default)
You want to see if the CA is configured to add the SID to certificates.

```
# Check the 'DisableExtensions' registry key on the CA
# If the SID OID (1.3.6.1.4.1.311.25.2) is NOT listed here, it is ENABLED (which is bad for the attacker).
certutil -config "CA-NAME\CA-SERVER" -getreg policy\DisableExtensions
```
#### 2. Disable the SID Extension (The Exploit)
You can use `certutil` to add the SID Extension OID to the "Disabled Extensions" list. This forces the CA to stop adding SIDs to new certificates.

```
# The OID for the SID Extension is 1.3.6.1.4.1.311.25.2
# We APPEND (+) this OID to the DisableExtensions list
certutil -config "CA-NAME\CA-SERVER" -setreg policy\DisableExtensions +1.3.6.1.4.1.311.25.2
```

#### 3. Restart the Service
Just like with ESC6, this change requires a service restart to take effect.
```
# Requires local admin or specific service rights on the CA
sc.exe \\CA-SERVER stop certsvc
sc.exe \\CA-SERVER start certsvc
```

### 💥 The Result

1. You (the attacker) request a certificate for `administrator@domain.local` using a standard template.
2. The CA issues the certificate **without** the SID (because you disabled it).
3. You use this certificate to authenticate.
4. The Domain Controller sees the missing SID, falls back to checking the UPN (`administrator@...`), and logs you in as the Administrator (ESC16 success).

### 💡 Summary Table

|**Attack**|**Permission**|**Registry Key Modified**|**Value Changed**|
|---|---|---|---|
|**ESC6**|`ManageCA`|`EditFlags`|**Enable** `EDITF_ATTRIBUTESUBJECTALTNAME2`|
|**ESC16**|`ManageCA`|`DisableExtension`|**Disable** OID `1.3.6.1.4.1.311.25.2`|

**Next Step:** Would you like to know how to check the **Domain Controller's** registry key (`StrongCertificateBindingEnforcement`) to see if it will actually accept these weak certificates?