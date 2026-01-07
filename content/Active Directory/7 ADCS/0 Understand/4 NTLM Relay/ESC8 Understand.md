
## ESC8: NTLM Relay to AD CS HTTP Endpoints
**ESC8** exploits the combination of two things:
1. The Certificate Authority (CA) has its **Web Enrollment** interface enabled.
2. This Web Enrollment interface uses **NTLM authentication** over **HTTP** (or HTTPS without proper protection), making it vulnerable to **NTLM Relay Attacks**.
###  The Core Misconfiguration

The vulnerability is rooted in the AD CS **Web Enrollment** role service, which, by default, often allows NTLM authentication without enforcing security features like **Extended Protection for Authentication (EPA)** or **SMB Signing/Channel Binding**.

If an attacker can trick a privileged machine (like a Domain Controller or a high-privilege server) into authenticating to them, they can **relay** that NTLM authentication to the CA's web endpoint. The CA sees the authentication request coming from a trusted party and issues a certificate **in the name of the victim machine/user**.

### The Attack Flow (The Kill Chain)
The attack is typically executed in four distinct phases:
#### Phase 1: Identify and Set Up the Listener
- **Identify Target:** The attacker confirms that the AD CS **Web Enrollment** interface (`/certsrv/`) is running and accepts NTLM authentication (often over HTTP on port 80 or 443 without EPA).
- **Set up Relay:** The attacker sets up a tool like `ntlmrelayx.py` to listen for incoming NTLM authentication attempts and automatically forward (relay) them to the vulnerable CA web endpoint, requesting a certificate.
#### Phase 2: Coerce Authentication (The Lure)
The attacker needs a high-value victim (e.g., a **Domain Controller**) to authenticate to their malicious listener. This is done through a technique called **NTLM Coercion**.
- **Attack Action:** Tools like **PetitPotam** or exploiting the **Print Spooler** service are used to force the Domain Controller to initiate an NTLM authentication connection to the attacker's listening machine.
#### Phase 3: The Relay and Enrollment
- **Relay:** When the Domain Controller attempts to authenticate to the attacker's host, the attacker's listener intercepts the NTLM challenge-response traffic and immediately relays it to the **CA's Web Enrollment** interface.
- **Certificate Issue:** The CA sees a legitimate NTLM authentication from the Domain Controller and, assuming a vulnerable template is available (like the `DomainController` template), it issues a certificate to the attacker in the name of the Domain Controller's computer account (e.g., `DC01$`).
#### Phase 4: Authentication and Domain Compromise
- **Authentication:** The attacker now possesses a valid certificate for the Domain Controller machine account. They use this certificate to perform **PKINIT** (Kerberos authentication) and get a Kerberos Ticket-Granting Ticket (TGT) for the DC.
- **Compromise:** With the Domain Controller's privileges, the attacker can perform a **DCSync** attack to retrieve the password hashes of all users, including the Domain Administrator, leading to a complete domain takeover.
###  Mitigation and Remediation
Mitigating ESC8 is critical and primarily involves disabling the underlying vulnerable protocols and services:
- **Enforce HTTPS/EPA:** If Web Enrollment is required, it must be hosted over HTTPS, and **Extended Protection for Authentication (EPA)** must be enabled and enforced. EPA binds the NTLM authentication to the TLS channel, making it impossible to relay.
- **Disable NTLM:** Disable NTLM authentication entirely on the **CA server** using Group Policy. This is the strongest defense.
- **Disable Web Enrollment:** If the service is not strictly needed (e.g., only auto-enrollment is used), disable the **Certificate Authority Web Enrollment** role service entirely.
- **Block Coercion:** Patch all known NTLM coercion vulnerabilities (e.g., Print Spooler, PetitPotam, DFSCoerce).
