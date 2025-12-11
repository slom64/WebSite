> [!question]
> i have seen something but i am not sure of it, imagine i have user that have SeBackup privilege in domain enviroment, and because of UAC i can't see that i have SeBackup. i have seen someone was able to enumerate such a privilege using something accessiable to all users. idk maybe something in SYSVOL or NETLOGON files.
> 

Yes — what you glimpsed is real: **Group Policy objects in SYSVOL are readable by Authenticated Users**, and the GPO files (specifically `GptTmpl.inf`) include the **User Rights Assignment** entries such as `SeBackupPrivilege`. So an unprivileged user can often enumerate which accounts/groups a domain GPO grants `SeBackupPrivilege` to simply by reading SYSVOL — no UAC bypass on the target machine required.

Below I’ll explain the idea, caveats, and give a ready-to-run PowerShell script that finds `SeBackupPrivilege` (and similar rights) across SYSVOL GPOs and resolves SIDs to names.

---
# How it works (short)

- Domain GPOs are stored under `\\<DOMAIN>\SYSVOL\<DOMAIN>\Policies\{GPO-GUID}\Machine\Microsoft\Windows NT\SecEdit\GptTmpl.inf`.
- That `GptTmpl.inf` contains a `[Privilege Rights]` section with lines like:
```
SeBackupPrivilege = *S-1-5-32-544,DOMAIN\SomeGroup
```
- `SYSVOL` is normally readable by Authenticated Users, so you can enumerate GPOs remotely and parse those files to see if `SeBackupPrivilege` was assigned to any account or group.
- UAC only affects _local privilege elevation_ on a host; it doesn’t stop you from reading SYSVOL over the network.
---
# Caveats / things to know
- GPOs can assign rights by SID (e.g. `S-1-5-21-...-512`) and those SIDs may refer to groups — you should resolve them to human-readable names.
- The **resultant effective** assignment on a particular DC may be influenced by multiple GPOs and local policy. Reading SYSVOL shows the GPO definitions, not the merged RSOP output on a particular machine.
- Some GPOs may be protected (read access restricted) by GPO permissions, but most orgs leave SYSVOL readable.
- Local `secedit.sdb` on a DC contains effective rights but is not readable by normal users (it's protected). So SYSVOL GPO parsing is the usual low-privilege path.