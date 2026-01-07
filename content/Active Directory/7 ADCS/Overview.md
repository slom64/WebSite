- ESC1: attacker can supply the `SubjectAltName`. `msPKI-Certificate-Name-Flag: ENROLLEE_SUPPLIES_SUBJECT` 
- ESC2: We have access to `AnyPurpos` certificate. which mean it has implicit `Enrollment agent OID`.
- ESC3: We have access to certificate that has `Enrollment agent` OID.
- ESC9: certificate that has `msPKI-Enrollment-Flag: CT_FLAG_NO_SECURITY_EXTENSION` which disable `szOID_NTDS_CA_SECURITY_EXT` make us fall back before certifried and request fake UPN.
- ESC10: if we have the registry keys `StrongCertificateBindingEnforcement = 0` or `CertificateMappingMethods = 0x4`, then we fall back before certifried, trusting UPN.
- ESC6: The CA can be vulnerable if a specific flag `EDITF_ATTRIBUTESUBJECTALTNAME2` is set. Which enable `SAN` for all templates make all templates vulnerable to `ESC1`. should combined with `ESC16`/`ESC9`.
- ESC4: If we have `FullControl`,`WriteDacl`, `WriteOwner` or specific `WriteProperty` rights over a template. Then we can modify the template to be vulnerable.
- ESC7: We have manageCA rights, which make us enable `ESC6 + ESC16` attack or enable `SubCA` certificate which is by default is vulnerable to ESC1.
- ESC5:
- ESC8: NTLM relay to http endpoint.
- ESC11: NTLM relay to rpc.
- ESC12: Extract private key of the CA or force the CA to sign certificate as administrator. Depends on how the private keys are stored.
- ESC13: **Issuance Policy with Privileged Group Linked**, Certificate is tied to policy that can give us privileges of other group.
- ESC14:  **Weak Explicit Certificate Mapping**. You can enumerate this using bloodhound.
- ESC15: **Arbitrary Application Policy Injection in V1 Templates (CVE-2024-49019 "EKUwu").**
- ESC16: CA disable SID extension for all certificates. **Security Extension Disabled on CA (Globally)**.


> [!Attention] 
> In modern systems. Having ESC6 alone is useless, and when we have ESC7 and enable ESC6 + ESC9/ESC16, having ESC6 is just optional and you still can do privilege escalation but with more steps "Modify account UPN before and after requesting certificate". ESC6 just make things faster and exploit it directly.
