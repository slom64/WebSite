
> [!Question] 
> What is the difference between `Cleint Authentication: True` and having `EKU: client authentication`.

It's Certipy's way of summarizing that the template can produce certificates valid for domain authentication scenarios.

The "`Client Authentication: True`" field in Certipy's output for a certificate template is a derived boolean flag indicating that the template is capable of issuing certificates usable for client authentication in the domain. This flag is set to `True` if the template's` Extended Key Usages` (EKUs) include any OID that enables authentication, such as:
- Client Authentication (OID: 1.3.6.1.5.5.7.3.2)
- Smart Card Logon (OID: 1.3.6.1.4.1.311.20.2.2)
- PKINIT Client Authentication (OID: 1.3.6.1.5.2.3.4)
- Any Purpose (OID: 2.5.29.37.0)
- Or if no EKUs are specified (which implies broad usage, often for subordinate CA templates)

In contrast, "EKU: Client Authentication" specifically refers to the presence of the Client Authentication OID (1.3.6.1.5.5.7.3.2) in the template's list of EKUs (found in the pKIExtendedKeyUsage attribute). This is the raw configuration detail, and Certipy displays the full list of EKUs (translated from OIDs to human-readable names) separately from the flag.

Both fields relate to the same overall purpose—enabling certificates for authentication (e.g., via Kerberos PKINIT for obtaining TGTs or SChannel for LDAPS)—but the flag is Certipy's interpreted summary of the template's authentication capability based on its EKUs, while the EKU entry is the explicit list of usages.
