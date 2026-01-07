
While **ESC8** targets the **HTTP Web Enrollment** service, **ESC11** targets the **RPC/DCOM Enrollment** service.
## ESC11: NTLM Relay to AD CS RPC/DCOM Endpoints

**ESC11** exploits a similar NTLM relay weakness, but instead of using the `/certsrv/` web interface, it targets the **Certificate Authority's Network Device Enrollment Service (NDES)** or the **RPC endpoint** used for standard auto-enrollment.

### The Core Vulnerability
The attack relies on the following conditions:
1. **NDES or RPC Enrollment is Enabled:** The Certificate Authority is running services that listen for enrollment requests over RPC/DCOM, which are vulnerable to NTLM relay.
2. **Weak NTLM Protection:** The network or the services are not enforcing security features like **Packet Privacy (signing/sealing)** or **Extended Protection for Authentication (EPA)**, which prevents the NTLM hash from being relayed.
3. **Vulnerable Certificate Templates:** The CA manages a certificate template that allows the attacker to specify an arbitrary Subject Alternative Name (SAN), usually due to the `EDITF_ATTRIBUTESUBJECTALTNAME2` flag being enabled (e.g., a misconfigured ESC6 template), **or** the attacker has leveraged **ESC7** to enable that flag globally.

### The Attack Flow
The goal of ESC11 is to trick a **privileged Domain Controller (DC)** into authenticating to the attacker, relay that authentication to the CA's RPC endpoint, and request a certificate with a spoofed identity.
#### Phase 1: Identify and Set Up the Listener
- **Identify Targets:** The attacker verifies that the CA's RPC/DCOM endpoints are accessible and that a vulnerable NTLM configuration is present.
- **Set up Relay:** The attacker sets up an NTLM relay tool (like `ntlmrelayx.py`) but configures the target relay service to be the **Certificate Enrollment RPC service** on the CA.
#### Phase 2: Coerce Authentication (The Lure)
This step is identical to ESC8. The attacker needs to coerce a **Domain Controller** or another high-privileged account to authenticate to the attacker's listener.
- **Attack Action:** Use NTLM coercion techniques (e.g., PetitPotam, Print Spooler) to force the DC to connect to the attacker's machine.
#### Phase 3: The Relay and Spoofed Enrollment
- **Relay:** When the DC connects and attempts to authenticate, the attacker intercepts the NTLM traffic and relays it to the CA's RPC enrollment service.
- **Spoofed Request:** Because the attacker is now authenticated as the Domain Controller _machine account_ on the CA, they use that session to request a certificate using a vulnerable template.
    - Crucially, they specify a malicious Subject Alternative Name (SAN) that identifies a high-privileged user (e.g., `administrator@domain.local`).
    - The CA issues the certificate for the **Administrator** because the request came from the highly trusted **Domain Controller** machine account, and the template/CA configuration allowed the SAN to be specified.
#### Phase 4: Authentication and Compromise
- **Authentication:** The attacker uses the newly issued certificate to perform **PKINIT** and authenticate as the targeted high-privileged user (Administrator).
- **Compromise:** The attacker is now authenticated as the Domain Administrator and can fully compromise the domain.
### ESC11 vs. ESC8 (The Key Difference)

| **Feature**              | **ESC8 (Web Enrollment)**                                                           | **ESC11 (RPC/DCOM Enrollment)**                                                             |
| ------------------------ | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| **Service Target**       | HTTP Web Enrollment (`/certsrv/`)                                                   | RPC/DCOM Enrollment (e.g., NDES)                                                            |
| **Protocol**             | HTTP/S (NTLM)                                                                       | RPC/DCOM (NTLM)                                                                             |
| **Template Requirement** | Requires a template configured for NDES or any template if SAN is allowed globally. | Often requires an **ESC6-vulnerable template** or the CA to be made vulnerable by **ESC7**. |
| **Mitigation Focus**     | Enforce EPA on IIS (Web Server).                                                    | Enforce **RPC/DCOM Packet Privacy** (Authentication Level).                                 |

### Mitigation and Remediation
- **Enforce RPC Packet Privacy:** This is the most effective mitigation. Ensure that **Packet Privacy (signing/sealing)** is required for RPC communication involving the CA and Domain Controllers. This prevents the NTLM hash from being relayed.
- **Use Protocol Transition:** Ensure that any Kerberos delegation used by the CA is properly configured and restricts the ability to impersonate arbitrary users.
- **Fix Template Misconfigurations:** Address the underlying ESC6 vulnerability by removing the `EDITF_ATTRIBUTESUBJECTALTNAME2` flag from all certificate templates.
- **Disable NTLM:** As with ESC8, disabling NTLM entirely on the CA server is the strongest defense.
